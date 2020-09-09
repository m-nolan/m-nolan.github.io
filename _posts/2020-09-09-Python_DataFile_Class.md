---
jupyter:
  jupytext:
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.2'
      jupytext_version: 1.6.0
  kernelspec:
    display_name: 'Python 3.7.7 64-bit (''ecog_is2s'': conda)'
    name: python_defaultSpec_1599612019232
---

# ML Signal analysis over many, many files (Pt. I: DataFile class)

Michael Nolan

2020.09.04.1948


### Problem
I'm trying to train a neural network to predict time series data (more on that in a later post). I need to access the data in order to do that. I've had some success accessing individual files in the past, loading the entire file's data into memory all at once. I can easily take that file and create a `torch.util.data.Dataset` from it, but that's not too scalable when files are ~2GB a piece and I'm working with 40+ files.

Previously I approached this problem by creating a dataset class that directly interfaced with the file with an unrefactored reading method. It is a little hacky, but works well for a single file. I also created a multifile class that can sample fixed-length sequences from a number of different data files through a single class interface. However, this is generally inflexible to different file types (I've created a few different pre-filtered versions of the same data) and gets the scope of file information (number of samples, channels, size of batch reads) spread across several class levels. I'd like a better interface to the data that's more flexible between different file encodings and 

### Solution
Classes all the way down. This requires stable and trackable access to data distributed over many many files. I think it makes sense to make a file class that holds data file, mask and channel information in an accessible form. All data loading will be through object instances of that class. A dataset class will then be made that is constructed from a collection of data file objects. This dataset will be very flexible w.r.t. sample sizes, allowing us to make arbitrary dataloaders to ease hyperparameter sweeps.

Anywho, let's go for it!

```python
# needed for class defs
import numpy as np
import scipy as sp
import os.path as path # may need to build a switch here for PC/POSIX
import json
import pickle as pkl

# not needed for class defs
import matplotlib.pyplot as plt
```

```python
# this is the file interface class. 
# - It will grab everything there is to know about the data file itself and it's supporting files (mask, experiment)
#   - Channel count
#   - Channel labels
#   - Filtering information
#   - Sampling rate
#   - Sample count (recording length)
# - It will provide a clean interface for reading data from arbitrary time points
#   - the argumentless call will read all data from the file
#   - arguments will change read start point and read length
#   - arguments will change the channels returned (masked v. full)
#   - arguments will toggle data masking (nan-pack recording)
class DataFile():

    def __init__(self, data_file_path, exp_file_path=None, mask_file_path=None):

        # parse file directory and components
        data_dir = path.dirname(data_file_path)
        data_basename = path.basename(data_file_path)
        rec_id, device_id, rec_type, data_ext = data_basename.split('.')

        # experiment data file: construct and load
        if not exp_file_path:
            exp_file_name = rec_id + 'experiment.json'
            exp_file_path = path.join(data_dir,exp_file_name)

        # mask file: construct and load
        if not mask_file_path:
            mask_file_name = rec_id + '.' + device_id + '.' + rec_type + '.mask.pkl'
            mask_file_path = path.join(data_dir,mask_file_name)
         
        # set recording parameters
        self.set_data_parameters(data_file_path,exp_file_path,mask_file_path)

    # this is returned when the print() command is called.
    def __repr__(self):
        path_repr_str = f'Data file object: {self.data_file_path}'
        sample_repr_str = f'\tsamples: {self.n_sample} ({self.n_sample/self.srate:0.2f}s, {self.data_mask.mean()*100:0.2f}% masked)'
        ch_repr_str = f'\tchannels: {self.n_ch} ({self.ch_idx.mean()*100:0.2f}% masked)'
        return path_repr_str + '\n' + sample_repr_str + '\n' + ch_repr_str
                

    # read data segment. Default call (no arguments) returns the entire recording.
    def read( self, t_start=0, t_len=-1, ch_idx=None, use_mask=True, mask_value=0., mask_pad_t=5 ):

        # get offset sample/byte values
        n_offset_samples = int(round(t_start * self.srate))
        n_offset_items = n_offset_samples * self.n_ch
        n_offset_bytes = n_offset_items * self.data_type().nbytes
        if t_len == -1:
            n_read_items = t_len
            n_read_samples = self.n_sample
        else:
            n_read_samples = t_len * self.srate
            n_read_items = n_read_samples * self.n_ch
        
        # read data
        with open(self.data_file_path,'rb') as f:
            data = np.fromfile(f,self.data_type,count=n_read_items,offset=n_offset_bytes).reshape(n_read_samples,self.n_ch).T

        # remove channels
        if not ch_idx:
            ch_idx = ~self.ch_idx
        data = data[ch_idx,:] # mask values are True for bad spots

        # mask data
        sample_idx = np.arange(n_offset_samples,n_offset_samples+n_read_samples)
        data[:,self.data_mask[sample_idx]] = mask_value

        # consider: time array? May not want to incorporate until global time is added
        return data


    # compute data parameter values and add as object attributes
    def set_data_parameters( self, data_file_path, exp_file_path, mask_file_path ):
        # parse file
        data_file = os.path.basename(data_file_path)
        data_file_kern = os.path.splitext(data_file)[0]
        rec_id, microdrive_name, rec_type = data_file_kern.split('.')
        data_path = os.path.dirname(data_file_path)
        
        # read experiment file
        exp_file = os.path.join(data_path,rec_id + ".experiment.json")
        with open(exp_file,'r') as f:
            exp_dict = json.load(f)
        
        # get microdrive parameters
        microdrive_name_list = [md['name'] for md in exp_dict['hardware']['microdrive']]
        microdrive_idx = [md_idx for md_idx, md in enumerate(microdrive_name_list) if microdrive_name == md][0]
        microdrive_dict = exp_dict['hardware']['microdrive'][microdrive_idx]
        electrode_label_list = [e['label'] for e in exp_dict['hardware']['microdrive'][0]['electrodes']]
        n_ch = len(electrode_label_list)
        
        # get srate
        if rec_type == 'raw':
            srate = exp_dict['hardware']['acquisition']['samplingrate']
            data_type = np.ushort
        elif rec_type == 'lfp':
            srate = 1000
            data_type = np.float32
        elif rec_type == 'clfp':
            srate = 1000
            data_type = np.float32
        
        # read mask
        ecog_mask_file = os.path.join(data_path,data_file_kern + ".mask.pkl")
        with open(ecog_mask_file,"rb") as mask_f:
            mask = pkl.load(mask_f)
        data_mask = mask["hf"] | mask["sat"]
        if 'ch' in mask.keys():
            ch_idx = mask['ch']
        else:
            ch_idx = np.arange(n_ch)
        
        # set parameters
        self.data_file_path = data_file_path
        self.exp_file_path = exp_file_path
        self.mask_file_path = mask_file_path
        self.rec_id = rec_id
        self.microdrive_name = microdrive_name
        self.rec_type = rec_type
        self.srate = srate
        self.data_type = data_type
        self.data_mask = data_mask
        self.n_ch = n_ch
        self.ch_idx = ch_idx
        self.ch_labels = electrode_label_list

        # set sample length information
        self.n_sample = len(self.data_mask)
        self.t_total = self.n_sample/self.srate # (s)

    # morphological transform: expand boolean array values a set distance from all current True values.
    # this is used to spread out mask estimates to more conservatively filter data files.
    def grow_bool_array( in_bool_array, growth_size=50.):
        None

```

That should do the trick! Let's build a tester to see how it works:

```python
# test class instance for the first recording in the dataset:
data_file_path = "E:\\aoLab\\Data\\WirelessData\\Goose_Multiscale_M1\\180325\\001\\rec001.LM1_ECOG_3.clfp.dat"
datafile = DataFile(data_file_path)
print(datafile)
t_start = 7000
t_len = 1
data_sample = datafile.read(t_start=t_start,t_len=t_len)
plt.plot(data_sample[11,:]);
plt.xlabel('time (s)')
plt.ylabel('amplitude ($\mu$V)')
plt.title('Sample data segment')
```

![signal_sample_image](images/20200909/ecog_sample.png?raw=true)

It works great. I even added a nice `self.__repr__()` method to format its output when passed to python's `print` function. 

Now let's get a sense of the read speeds. I'll repeat the `datafile.read()` call with random time points 1000 times and get some statistics on average loading time.

```python
import time
n_iter = 1000
t_len = 1
t_iter = np.zeros(n_iter)
for k in range(n_iter):
    t_start = np.random.rand()*(datafile.t_total-t_len)
    t = time.time()
    data = datafile.read(t_start=t_start,t_len=t_len)
    t_iter[k] = time.time()-t
print(f'{n_iter} data pulls complete.')
```

```python tags=[]
plt.hist(t_iter,100,label='read times');
plt.axvline(t_iter.mean(),color='r',label='mean')
plt.xscale('linear')
plt.xlabel('Read time (s)')
plt.ylabel('sample frequency')
plt.title('1s sample read time distribution - DataFile class')
plt.legend(loc=0)
print(f'Mean read time: {t_iter.mean():0.3E}')
print(f'Total read time ({n_iter} reads): {t_iter.sum():0.3E}')
```

![read time histogram](images/20200909/read_time_hist.png?raw=true)

Neat! I'm unsure of what's causing the split distribution in read times here, but we're getting a very, very fast read speed for >80% of draws. 

The total batch read time is 0.16s. If we're to use this as the data access API for a pytorch dataloader, we might want something faster. This file is currently on an SSD connected to my computer through a USBC/3.0 port. Moving all of this data to my system's internal memory may provide me with an immediate speed-up, [provided that I use my PCIe SSD.](https://www.unbxtech.com/2019/03/pcie-sata-usb-interfaces-explained.html). Using different data reading methods to optimize the code [may not lead to much significant improvement, so I'll leave them be for now.](http://rabexc.org/posts/io-performance-in-python).

Either way, it all works great! This will make the next step a bit easier: creating a dataset interface to a list of these DataFile objects. Coming soon!
