
# Bitcoin Private Key Finder Script - Puzzle 66

This Python script is designed to help solve Bitcoin-related puzzles by searching for a private key within a specified range. The script leverages parallel processing to efficiently utilize all available CPU cores, and it includes optional randomization to enhance the chances of finding the key faster.

## How It Works

- **Parallel Processing**: The script divides the key range across multiple CPU cores, allowing simultaneous checking of different portions of the range.
- **Randomization**: By enabling randomization, the script selects keys randomly within the specified range, potentially increasing the chances of an early find.
- **Progress Feedback**: The script provides real-time feedback on its progress, showing how many keys have been checked and the percentage of completion.

## Prerequisites

Ensure that you have the necessary Python package installed:

```bash
pip install bitcoin
```

## Usage

Here's the script (`puzzle66.py`):

```python
import bitcoin
from bitcoin import privtopub, pubtoaddr
import multiprocessing
import random
import sys

# Function to check a range of keys, with randomization by sampling
def check_key_range(start_key, stop_key, target_address, process_id, randomize=False):
    key_range_size = stop_key - start_key + 1
    
    checked_keys = set()  # To avoid duplicate checks
    
    for i in range(key_range_size):
        if randomize:
            key = random.randint(start_key, stop_key)  # Random key within the range
            while key in checked_keys:
                key = random.randint(start_key, stop_key)  # Ensure no duplicates
        else:
            key = start_key + i
        
        checked_keys.add(key)
        priv_key_hex = format(key, '064x')
        pub_key = privtopub(priv_key_hex)
        generated_address = pubtoaddr(pub_key)
        
        # Progress feedback for this process
        if i % 1000 == 0:  # Adjust frequency as needed
            progress = (i / key_range_size) * 100
            print(f"Process {process_id}: Checked {i}/{key_range_size} keys ({progress:.4f}% complete)")
            sys.stdout.flush()

        if generated_address == target_address:
            print(f"Private key found by Process {process_id}: {priv_key_hex}")
            sys.stdout.flush()
            return priv_key_hex
    return None

if __name__ == '__main__':
    target_address = "13zb1hQbWVsc2S7ZTZnP2G4undNNpdh5so"
    start_key_hex = "0000000000000000000000000000000000000000000000020000000000000000"
    stop_key_hex = "000000000000000000000000000000000000000000000003ffffffffffffffff"
    
    start_key = int(start_key_hex, 16)
    stop_key = int(stop_key_hex, 16)
    
    # Determine the number of processes to run in parallel
    num_processes = multiprocessing.cpu_count()  # Use all available CPU cores
    key_range = stop_key - start_key + 1
    range_per_process = key_range // num_processes
    
    processes = []
    for i in range(num_processes):
        range_start = start_key + i * range_per_process
        range_stop = start_key + (i + 1) * range_per_process - 1 if i != num_processes - 1 else stop_key
        p = multiprocessing.Process(target=check_key_range, args=(range_start, range_stop, target_address, i + 1, True))
        processes.append(p)
        p.start()
    
    for p in processes:
        p.join()
    
    print("Search completed.")
```

### How to Run the Script

1. **Save the Script**: Copy the script into a file named `puzzle66.py`.
2. **Run the Script**: Use Python to execute the script in your terminal or command prompt:

   ```bash
   python3 puzzle66.py
   ```

3. **Monitor Progress**: The script will output the progress of each process, showing how many keys have been checked and the percentage complete. If a private key is found, it will be displayed.

### Example Output

When running the script, you might see output similar to this:

```
Process 1: Checked 1000/1000000 keys (0.1000% complete)
Process 2: Checked 1000/1000000 keys (0.1000% complete)
Process 1: Checked 2000/1000000 keys (0.2000% complete)
Process 2: Checked 2000/1000000 keys (0.2000% complete)
Private key found by Process 1: 00000000000000000000000000000000000000000000000200000000000000ff
Search completed.
```

In this example, Process 1 successfully found the private key. The script then completes its execution.

## Donations

If you find this script helpful, consider making a donation to support further development and to assist others in solving similar puzzles.

- **BTC**: `14cxycAfEb3JuFjETXMcyyk56YkDA8DHgT`
- **ETH**: `0xc040ddc63d621cfe9f539f4758cd0d805581e24e`
- **TRX**: `TKojsCqXzdqsLPAMcAiA5gfQSrfGBREwvV`
- **SOL**: `6bBrtq9PCBwbofFcofLaehPvMZ3bQFuoRzkmjXfANq7V`

Your support is greatly appreciated!
