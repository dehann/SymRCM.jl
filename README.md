[![Project Status: Active – The project has reached a stable, usable state and is being actively developed.](http://www.repostatus.org/badges/latest/active.svg)](http://www.repostatus.org/#active)
[![Build Status](https://img.shields.io/travis/PetrKryslUCSD/SymRCM.jl/master.svg?label=Linux+MacOSX+Windows)](https://travis-ci.org/PetrKryslUCSD/SymRCM.jl) 
[![Coverage Status](https://coveralls.io/repos/github/PetrKryslUCSD/SymRCM.jl/badge.svg?branch=master)](https://coveralls.io/github/PetrKryslUCSD/SymRCM.jl?branch=master) 

# SymRCM: Reverse Cuthill-McKee node-renumbering algorithm for sparse matrices.

`SymRCM` is a tiny package for computing the Reverse Cuthill-McKee node permutation from a sparse matrix. The goal is to minimize the "profile" of the matrix, usually aimed at reducing the cost of LDLT or Choleski factorizations.

## Get SymRCM

This package is  registered, and hence one can do just
```julia
] add SymRCM
```
Only version 1.x and the nightly builds of Julia are supported.

## Testing

```julia
] test SymRCM
```

## Usage

For a sparse matrix `A` the basic usage is:
```julia
p = symrcm(A)
```
To solve the system of linear algebraic equations `x = A * b` with renumbering one can do
```
using SparseArrays
using LinearAlgebra
n = 7;
A = sprand(n, n, 1/n)
A = A + A' + 1.0 * I
A = sparse(A)
b = rand(n)
using SymRCM
p = symrcm(A)
ip = similar(p) # inverse permutation
ip[p] = 1:length(p)
xp = A[p, p] \ b[p] # solution to the renumbered system of equations
x = xp[ip] # solution to the original system of equations
A \ b # this is the direct solution which should be identical to the above
```

Lower-level functions may also be useful. The adjacency graph and the node degrees may be calculated as
```
ag = adjgraph(A; sortbydeg = true)
nd = nodedegrees(ag)
```
which can be used to compute the renumbering as
```
numbering1 = symrcm(ag, nd) # using the lower-level functions
numbering2 = symrcm(A) # direct use of the sparse matrix
```
and these two will be identical.

For significantly populated matrices the sorting of the neighbor lists
may be a significant expense. In that case the sorting may be turned off.
```
p = symrcm(A; sortbydeg = false) # note the keyword argument
```
Very often the resulting permutation is as good  as if the lists were sorted.

## Performance
 
Relative numbers may be of interest:

Present package code
```
using SymRCM                    
using SparseArrays                                     
let   
    S = sprand(10000000, 10000000, 0.0000001);    
    S = S + transpose(S);                                                       
    @time p = symrcm(S; sortbydeg = true);  
    I, J, V = findnz(S[p, p])
    @show bw = maximum(I .- J) + 1            
    @time p = symrcm(S; sortbydeg = false);   
    I, J, V = findnz(S[p, p])
    @show bw = maximum(I .- J) + 1     
    true   
end;  
```
produces
```
  9.271877 seconds (10.07 M allocations: 1.474 GiB, 6.42% gc time)          
bw = maximum(I .- J) + 1 = 1535838              
  7.189242 seconds (10.00 M allocations: 1.471 GiB, 3.04% gc time)    
bw = maximum(I .- J) + 1 = 1535722   
```

Matlab 2020a:
```
>> S = sprand(10000000, 10000000, 0.0000001);     
S = S + S';
tic; p = symrcm(S); toc  
[i,j] = find(S(p,p));
bw = max(i-j) + 1
Elapsed time is 9.258979 seconds.
bw =
     1535477
```
