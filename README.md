# Distributed Hash Tables Using Chord For NFL Football

## Overview

This project implements a Chord Distributed Hash Table (DHT) system using Python 3, based on the original Chord protocol described by Stoica, Morris, Karger, Kaashoek, and Balakrishna in their 2001 paper. The system is designed to efficiently store and query passing statistics data from the National Football League (NFL). The Chord protocol ensures consistent, distributed data storage and provides a mechanism for nodes to join and leave the network while maintaining correct finger tables.

This implementation adheres to the Chord system as outlined in Section 4 of the paper, which requires updating the finger tables when a new node joins the network. This does not include the enhancements described in Section 5.

## File Structure

- `chord_node.py`: Handles the creation and management of individual nodes within the Chord network.
- `chord_populate.py`: Populates the Chord network with data from the NFL passing statistics dataset.
- `chord_query.py`: Allows users to query or add key-value pairs to the Chord DHT.

## Key Concepts

### Chord Network
- **Node**: A node in the Chord system is identified by a unique identifier generated using SHA-1, applied to the node's endpoint (including its port number).
- **Finger Table**: Each node maintains a finger table, which is used to efficiently locate the successor of a key in the system. This table is updated whenever a new node joins or leaves the network.
- **Key-Value Storage**: The key used for storing NFL data is derived from concatenating the `playerid` and `year` columns of the dataset and applying SHA-1 hashing.

### System Behavior
1. **Joining a Node**: A new node can join an existing Chord network, at which point its finger table is updated. The node will also take ownership of the appropriate keys and associated data.
2. **Querying a Key**: Any node in the network can be queried for a specific key. The system will find the node that owns the key and return the associated value.
3. **Inserting a Key**: A new key-value pair can be inserted into the network, replacing any existing value for that key.

### Data Storage
The data being stored is NFL passing statistics, with each row represented by a key-value pair. The key is computed from the `playerid` and `year` columns using SHA-1, and the value is stored as a dictionary containing the other columns from the row.

## How to Use

### Prerequisites
- Python 3.x
- Libraries: `socket`, `hashlib`, `pickle`, `threading`

### Files

1. **`chord_node.py`**:
   - Command Line Argument: Port number of an existing node (or 0 for a new network).
   - Behavior: Creates a node in the network, joins the network if needed, and listens for incoming connections from other nodes or queriers.

   Example:
   ```bash
   python chord_node.py 8080
   ```

2. **`chord_populate.py`**:
   - Command Line Arguments:
     - Port number of an existing node.
     - Filename of the data file (e.g., NFL passing statistics CSV file).
   - Behavior: Loads data from the provided file into the Chord network.

   Example:
   ```bash
   python chord_populate.py 8080 nfl_passing_data.csv
   ```

3. **`chord_query.py`**:
   - Command Line Arguments:
     - Port number of an existing node.
     - Key (a string derived from `playerid` and `year` in the file).
   - Behavior: Queries the Chord network for the value associated with the provided key.

   Example:
   ```bash
   python chord_query.py 8080 "playerid_year"
   ```

### SHA-1 Hashing
- The system uses SHA-1 to hash keys and node identifiers. For testing, you may truncate the 160-bit hash to a smaller number of bits using modulo arithmetic.

### ModRange Class
The `ModRange` class is used to handle ranges that wrap around the 0 node using modulo arithmetic. This is useful for handling the circular nature of the Chord system.

```python
class ModRange:
    def __init__(self, start, stop, divisor):
        self.divisor = divisor
        self.start = start % self.divisor
        self.stop = stop % self.divisor
        if self.start < self.stop:
            self.intervals = (range(self.start, self.stop),)
        elif self.stop == 0:
            self.intervals = (range(self.start, self.divisor),)
        else:
            self.intervals = (range(self.start, self.divisor), range(0, self.stop))
```

### ChordNode Class
The `ChordNode` class represents a node in the Chord network. Each node manages its own finger table, handles the joining of new nodes, and can locate a key's successor.

```python
class ChordNode:
    def __init__(self, n):
        self.node = n
        self.finger = [None] + [FingerEntry(n, k) for k in range(1, M+1)]
        self.predecessor = None
        self.keys = {}

    def find_successor(self, id):
        np = self.find_predecessor(id)
        return self.call_rpc(np, 'successor')
```

### Joining a Node
When a new node joins the network, it updates its finger table, and the system ensures that all affected nodes update their finger tables as well.

```python
def update_others(self):
    for i in range(1, M+1):
        p = self.find_predecessor((1 + self.node - 2**(i-1) + NODES) % NODES)
        self.call_rpc(p, 'update_finger_table', self.node, i)
```

### Handling RPCs with Threads
To avoid deadlocks when handling multiple RPC requests, the system uses threading to manage simultaneous connections.

```python
def handle_rpc(self, client):
    rpc = client.recv(BUF_SZ)
    method, arg1, arg2 = pickle.loads(rpc)
    result = self.dispatch_rpc(method, arg1, arg2)
    client.sendall(pickle.dumps(result))
```

## Conclusion
This Chord DHT implementation adheres to the guidelines set in the original Chord paper and allows for efficient distributed storage and querying of NFL passing statistics. The system ensures that finger tables are correctly updated when nodes join, and supports querying and adding data using a simple interface.
