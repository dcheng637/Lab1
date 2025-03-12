Here is the code for the coordinator:

import socket
import pickle
import collections

def send_chunk_and_get_counts(worker_host, worker_port, text_chunk):
    """
    Connect to a worker, send the text chunk, and retrieve the word-count dictionary.
    """
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((worker_host, worker_port))
    s.sendall(pickle.dumps(text_chunk))  # Send text chunk
    data = s.recv(4096)                  # Receive partial counts
    s.close()
    return pickle.loads(data)            # Return dictionary

def main():
    # Define workers (host, port). I added a third worker.
    workers = [
        ("127.0.0.1", 5001),
        ("127.0.0.1", 5002),
        ("127.0.0.1", 5003)
    ]

    # A sample text for demonstration (you can replace or read from file)
    text = """
    hello professor
    this is Danny's version of the code
    hello again professor it's me Danny
    distributed computing is fine
    my favorite soda is coke
    my favorite fruit is banana
    """

    # Split text into lines
    lines = text.strip().split('\n')

    # Splitting the work so that 3 chunks can do stuff. There's probably a better way to do this more dynamically.

    

    chunk1 = "\n".join(lines[:len(lines) // 3])
    chunk2 = "\n".join(lines[len(lines) // 3: 2 * len(lines) // 3])
    chunk3 = "\n".join(lines[2 * len(lines) // 3:])
    

    chunks = [chunk1, chunk2, chunk3]

    # Aggregate results here
    combined_counts = collections.Counter()

    # Distribute each chunk to corresponding worker
    for i, (host, port) in enumerate(workers):
        result_dict = send_chunk_and_get_counts(host, port, chunks[i])
        combined_counts.update(result_dict)

    # Print final word counts
    print("[Coordinator] Final Word Counts:")
    for word, count in combined_counts.items():
        print(f"{word}: {count}")

    # Send "DONE" to each worker so they shut down
    for (host, port) in workers:
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.connect((host, port))
        s.sendall(pickle.dumps("DONE"))
        s.close()

if __name__ == "__main__":
    main()
