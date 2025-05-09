# CS 179: GPU Computing
## Assignment 4

**Due:** Wednesday, April 30, 2025

### Submission:
---

Instead of emailing us the solution, put a zip file in your home directory
on the lab machine, in the format:

`lab4_2025_submission.zip`

Your submission should be a single archive file (.zip)
with your README file and all code.

### Resource usage:
---

You'll be using the CUBLAS and cuSolver APIs, so feel free to look up general documentation.
Also, feel free to use an online obj file visualizer to check your program correctness, as
explained below.

## Point-Cloud Alignment
---

### Introduction:
---

Similarly to lab3 this lab will make use of a CUDA library. We will be using cuBLAS to accelerate
basic linear algebra operations. There are several TODO comments in point_alignment.cu, and your
assignment is to fill those in.

### Background: obj files
---
The obj file format is commonly used to represent 3D models (and if you've taken CS 171, you're
probably very familiar with this file format).  Don't worry if you've never seen this file format
before, though, because we provide the code to process an input model into a single float array
representing the vertex coordinates.

### Background: point cloud alignment
---
As explained in class, the task is to 'align' two different sets of points to minimize the
overall 'misalignment' between the two sets.  Another way to think about this is, given one
3D mesh, and another 3D mesh that has been rotated, scaled, and translated, plus had some
noise added, you want to recover the transformation matrix that represents the rotation, scaling,
and translation operations.

Once you've filled in all of the TODOs, you can check the correctness of the output in two ways:
- directly check the transformation matrix that you solved for (which will be printed out)
- view the 3D model in the output obj file to see if it matches with the input obj file (part of
  your task is to apply the transformation matrix that you found to the original model, and if
  you've done everything right, this should transform the first object into the second).  Most
  computers will already have ways to visualize obj files (for example, Finder/Preview on Mac
  both will visualize obj files automatically) but there are also plenty of resources available
  online that will allow you to view the obj file.  One example is https://3dviewer.net

### Running the code
---
Usage: `./point_alignment [file1.obj] [file2.obj] [output.obj]`

We recommend using the models found in the resources folder to test your code.
Specifically, `resources/bunny2_trans.obj` `resources/bunny2.obj` should both give clean results.

Note: In order to align two point clouds, they must have the same number of vertices.

### To do:
---

See all of the TODO comments in point_alignment.cu
- initialize CUBLAS handle
- allocate device memory & initialize and copy from host properly
- calculate xx4x4 and x2Tx1 matrices using CUBLAS, as described in the comments
- solve the system using cuSolver
- copy the resulting transformation matrix back to the host
- apply the transformation matrix to the original model (and all necessary setup)
- free all resources

### Notes:
---

Make sure to think about column vs row major formats of storing matrices.  In this assignment,
the code that reads in the obj file stores the vertices in a  flattened float array with the format:
`{x1, x2, x3,..., y1, y2, y3, ..., z1, z2, z3, ..., 1, 1, 1}`
The added 1 is for homogenous coordinates and the purpose of working with the transformation matrices.
The CUBLAS function calls take in parameters that specify whether or not to take the transpose of
one of the input matrices, so make sure to consider the data format as you write your CUBLAS calls.
When the calls read from the stored data, they process it like this:
```
           point_dim (= 4)
            [x1 y1 z1 1]
            [x2 y2 z2 1]
num_points  [x3 y3 z3 1]
            [x4 y4 z4 1]
            [x5 y5 z5 1]
            [x6 y6 z6 1]
```

However, for our array float to object function, the code expects a float array with the format:
`{x1, y1, z1, 1, x2, y2, z2, ....., xn, yn, zn, 1}`
This is due to multiplying the transformation matrix to transpose of vertex array.
Specifically, our transformation matrix is 4x4, and so we need to multiply
it by a 4xnum_points matrix (which will be vertex_array^T), so the result
will be a 4xnum_points matrix, which will look like:
```
                num_points
           [x1 x2 x3 x4 x5 x6]
           [y1 y2 y3 y4 y5 y6]
point_dim  [z1 z2 z3 z4 z5 z6]
           [1  1  1  1  1  1 ]
```

And reading this in column major format, it looks like
`{x1, y1, z1, 1, x2, y2, z2, ....., xn, yn, zn, 1}`

---

Assignment written for Spring 2019 by George Stathopoulos, Jenny Lee, and Mary Giambrone. 