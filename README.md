# DeMu
This is an algorithm that demutabilize code in [EO language](https://github.com/objectionary/eo)

It has an ``.eo`` (or ``.xmir``) files as input and returns ``.eo`` (or ``.xmir``) file as output.

Setup:
```
sudo apt-install rust
```
Compile:
```
git clone https://github.com/MCJOHN974/DeMu
cd DeMu
make
```
Use:
```
./demu <eo program>
```

# Algorithms

Our program consists on several independent checks, they work in different cases. Together they can remove a significant part of mutable objects.

## Loop to recursion
First algorithm find loops with one mutable object---iterator and replace this loop to recursion without mutable objects. Example of that loop:
```
[n] > loop
  memory 0 > index
  while > @
    index.less n
    seq
      sprintf
        "%d\n"
        index
      index.write index.plus 1
```

This loop we want rewrite the following way:

```
[n, index] > loop_helper
  if > @
    index.less n
    seq
      sprintf
        "%d\n"
        index
      loop_helper n (index.plus 1)
      
[n] > loop_demu
  loop_helper > @
    n
    0
```
Syntax in the objects can be deprecated, but I think you got the idea (of course will be fixed)}
Here we almost did not change the code of ``eq`` object. 
Steps of this algorithm:
    1)  indicate that object has mutable attribute (simple check if exactly one memory attribute exists in the code) \\
    2) indicate that mutable attribute used as an index in the loop.  (check that there exists while loop, condition of while loop contains comparing with the mutable attribute, and .write called exactly one time, in the body of the loop
    3) add n as second free attribute, remove mutable attribute, change while to if and .write to calling next level of recursion.
    4) create an final object (loop\_demu in the example) that just decorates recursion with correct start parameters
It will be good to think about formal proof of it in the future
