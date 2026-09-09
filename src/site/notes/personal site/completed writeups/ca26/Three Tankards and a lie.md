---
{"dg-publish":true,"permalink":"/personal-site/completed-writeups/ca26/three-tankards-and-a-lie/","tags":["CyberApocalypse-26","coding","python"],"dg-note-properties":{"tags":["CyberApocalypse-26","coding","python"]}}
---

# Problem:
```
Three Tankards and a Lie
N tankards stand on the bar, numbered 1..N; tankard i starts holding
item i. M swaps are performed in order, each exchanging the contents
currently sitting at two named positions. For each of Q queries (a
starting position p), report the final position of the item that
started at position p, after all M swaps have been applied.

Line 1: N M Q
Next M lines: a b   (positions swapped, 1-indexed)
Next Q lines: p      (a starting position to track)

1 <= N <= 2000
0 <= M <= 5000
1 <= Q <= 2000
1 <= a, b <= N
1 <= p <= N

Example Input:
5 4 2
1 3
2 4
3 5
4 1
3
5

Expected output:
4
3
```
# Code:
*maybe not the best way to solve this, but how I did it*
```python
# take in the params
params = input().split()
params = [int(x) for x in params]
N = params[0]
M = params[1]
Q = params[2]
# make a map size of N
tankards = {i: i for i in range(1, N+1)}# (tankard number, contents)
#print(tankards)

# track moves/swaps
for i in range(M):
    move = input().split()
    move = [int(x) for x in move]
    #print(move)
    tankards[move[0]], tankards[move[1]] = tankards[move[1]], tankards[move[0]]
    #print(tankards)

# get contents
for i in range(Q):
    tankard = input()
    # I need to invert this, cause it wants the cup that has the values, not the value of those cups. I misinterpreted
    tankards_inv = {value: key for key, value in tankards.items()}
    print(tankards_inv[int(tankard)])
```

