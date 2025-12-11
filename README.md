# Hardware_Sorter

The algorithm is : 
It is serialized using a shift register, after which a sorting node queue is proposed. The queue is made up of pairs of nodes li (left) and ri (right). In each cycle, a value pi enters the node pair from the left, and it decides the following: 
• If li is empty, then Li ← Pi. 
• If Pi ≤ Li, then Ri ← li and Li ← Pi. 
• If Pi > Li, then Ri ← Pi. 
• Ri is always pushed to the right as Pi+1 In this way, the lowest and newest values are kept at the leftmost nodes.

When all values have been inputted, we can be sure that l0 contains the lowest value, while the rest of the values are partially sorted. At this point, a signal activates that flips the behavior. Nodes now output to the left instead of the right, with li and ri exchanging jobs. Now ri will keep the highest input value, while pushing lower values to li−1. This ensures that the output is ordered. However, in the event of ties, this approach favors the latest inputs.

The total cost for this step is N cycles to fill the queue, and another N to empty it, for a total of O(2N).

## Implementation 

### Sort Node (sort_node_act.sv)

This is typically a compare-swap element. It performs:

Core Function
<pre> ```if (a > b)  
    swap a and b  
else  
    leave them unchanged ``` </pre>


And it also propagate index along with data:

<pre> ```if (data[i] > data[j])
    swap(data[i], data[j])
    swap(index[i], index[j]) ``` </pre>


This allows you to output sorted indices instead of sorted values.

Parameters Usually Inside sort_node_act

Common parameters include:

#### Parameter	Meaning
DATA_W	Bit-width of each input data word. <br>
IDX_W	Bit-width of the index value. <br>
SIGNED / UNSIGNED	Whether values are interpreted as signed integers. <br>
PIPELINE	Whether compare-swap is combinational or pipelined. <br>

Function:
A sort node takes two inputs, performs the compare-swap, and outputs the sorted pair. This is the only building block required for most hardware sorting networks.

### Serial Sorter (serial_sorter.sv)

This is the top-level architecture controlling the sort nodes.

Its role is to:

- Load N input values (and indices).
- Iterate through a bubble-sort-like algorithm.
- Use one compare-swap node per cycle, addressing the array sequentially.
- After enough passes, output all sorted values and indices.

#### Parameter	Meaning 
NUM_ELEM	Number of values to be sorted. <br>
DATA_W	Width of each probability / data value. <br>
IDX_W	Width of the index tag. <br>
SORT_ASC / SORT_DESC	Sorting direction (if implemented). <br>
PIPE_STAGES	Number of pipeline stages for the node. <br>
LOOP_COUNT	Number of compare-swap rounds needed for sorting (generally NUM_ELEM - 1). 
