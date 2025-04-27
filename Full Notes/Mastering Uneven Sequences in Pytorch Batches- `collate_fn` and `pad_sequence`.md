26-04-2025 11:51
Status: #unrefined 
Tags: [[Natural Language Processing]] [[Pytorch]] [[Deep Learning]]


# Handling Variable-Length Sequences in PyTorch: A Beginner's Guide to `collate_fn` and `pad_sequence`

When working with real-world data like text, audio, or time series, one common issue is that your input sequences are of variable lengths. This can become a headache when trying to batch your data for training. Thankfully, PyTorch provides tools like `collate_fn` and `pad_sequence` that can help.

```python
from torch.nn.utils.rnn import pad_sequence
from torch.utils.data import DataLoader
```

This blog will walk you through the absolute basics of what these functions do, why you need them, and how to use them with examples.

---

## Why Do We Need `collate_fn` and `pad_sequence`?

### Problem: Sequences Are Not Always the Same Length

Imagine you're building a language model, and your dataset consists of sentences like:

```python
["Hello world"]
["How are you?"]
["Good"]
```

When converting these to tensors, you'll end up with sequences of different lengths. However, PyTorch's `DataLoader` expects all samples in a batch to be the same size. This is where things break down:

- Default DataLoader will try to stack them, which will raise an error.
- RNNs need to process sequences in batches, so you can't just ignore batching.

### Solution: Custom Collation + Padding

To handle this, we do two things:

1. **Pad the sequences to the same length** using `pad_sequence()`.
2. **Override the default batching behavior** using `collate_fn` in `DataLoader`.

---

## Understanding `pad_sequence`

### What is `pad_sequence`?

`pad_sequence` is a utility function from `torch.nn.utils.rnn` that pads a list of tensors (e.g., sequences) to the same length with a specified value (default is 0).

```python
from torch.nn.utils.rnn import pad_sequence

sequences = [
    torch.tensor([1, 2, 3]),
    torch.tensor([4, 5]),
    torch.tensor([6])
]

padded = pad_sequence(sequences, batch_first=True, padding_value=0)
print(padded)
```

### Output:

```
tensor([
    [1, 2, 3],
    [4, 5, 0],
    [6, 0, 0]
])
```

- `batch_first=True` ensures the shape is `(batch_size, seq_length)`. If `batch_first=False` ensures the shape is `(seq_length, batch_size)`
- `padding_value=0` means missing elements will be filled with 0. You can also pad the elements with `vocab['<pad>']` if you have a special index for padding in your vocabulary.

---

## Understanding `collate_fn`

### What is `collate_fn`?

When you create a `DataLoader`, you can pass a custom `collate_fn` that tells PyTorch how to combine a list of samples into a batch.

Default behavior just tries to stack tensors directly:

```python
DataLoader(dataset, batch_size=2)
```

This works for fixed-size inputs but fails for variable-length ones. It will also give you error when the batch size would not be rectangular.

Instead, we use:

```python
DataLoader(dataset, batch_size=2, collate_fn=custom_collate_fn)
```

### Example `collate_fn`

```python
def collate_fn(batch):
    return pad_sequence(batch, batch_first=True, padding_value=0)
```

Now the DataLoader will pad sequences in each batch automatically.

In Neural Machine Translation (NMT) cases the dataset will return a sample of data in batch like this:

```python
(src_tensor, tgt_tensor)
```

Where the `src_tensor` can be encoded French sentence and the `tgt_tensor` can be encoded English sentence.

For example the batch will look like this:

```python
batch = [
    (tensor([5, 9, 2]), tensor([3, 8, 7, 2])),
    (tensor([4, 7, 8, 2]), tensor([3, 2])),
    ...
]
```

Now, to handle this complexity, you have the flexibility to customize your `collate_fn` like this:

```python
def collate_fn_nmt(batch):
	src_batch, tgt_batch = zip(*batch)
	'''
	zip unpacks and regroups
	src_batch = (tensor([5, 9, 2]), tensor([4, 7, 8, 2]), ...)
	trg_batch = (tensor([3, 8, 7, 2]), tensor([3, 2]), ...)
	'''
	padded_src_btch = pad_sequence(src_batch, batch_first=True, padding_value=vocab_src['<pad>'])
	padded_tgt_btch = pad_sequence(tgt_batch, batch_first=True, padding_value=vocab_tgt['<pad>'])
	return padded_src_btch, padded_tgt_batch

```

---

## Putting It All Together

### Custom Dataset

```python
from torch.utils.data import Dataset

class MyDataset(Dataset):
    def __init__(self):
        self.data = [
            torch.tensor([1, 2, 3]),
            torch.tensor([4, 5]),
            torch.tensor([6])
        ]

    def __len__(self):
        return len(self.data)

    def __getitem__(self, idx):
        return self.data[idx]
```

### DataLoader with Custom `collate_fn`

```python
from torch.utils.data import DataLoader

dataset = MyDataset()
loader = DataLoader(dataset, batch_size=2, collate_fn=collate_fn)

for i, batch in enumerate(loader):
    print(f"Batch {i}:")
    print(batch)
```

### Output:

```
Batch 0:
[[1, 2, 3],
 [4, 5, 0]]

Batch 1:
[[6, 0, 0]]
```

---

## Bonus: Returning Sequence Lengths

For RNNs, you often need the original sequence lengths. Here's how you can return both padded sequences and lengths:

```python
def collate_fn_with_lengths(batch):
    lengths = torch.tensor([len(seq) for seq in batch])
    padded = pad_sequence(batch, batch_first=True, padding_value=0)
    return padded, lengths
```

Use it in the DataLoader:

```python
loader = DataLoader(dataset, batch_size=2, collate_fn=collate_fn_with_lengths)

for padded, lengths in loader:
    print("Padded:", padded)
    print("Lengths:", lengths)
```

Output:

```
Padded: tensor([[1, 2, 3],
                [4, 5, 0]])
Lengths: tensor([3, 2])

Padded: tensor([[6, 0, 0]])
Lengths: tensor([1])
```

---

## Summary

- PyTorch's `DataLoader` expects uniform input shapes, which breaks when working with sequences of varying lengths.
- Use `pad_sequence` to pad sequences to the same length.
- Use `collate_fn` in the DataLoader to customize how samples are batched.
- For RNNs, keep track of the original lengths for packing and masking.

These tools make PyTorch flexible and powerful for handling sequence data.

References:
- [PyTorch `pad_sequence`](https://pytorch.org/docs/stable/generated/torch.nn.utils.rnn.pad_sequence.html)
- [PyTorch `DataLoader` and `collate_fn`](https://pytorch.org/docs/stable/data.html#dataloader-collate-fn)
