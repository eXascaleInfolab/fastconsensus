# Fast Consensus Clustering in Networks

Fast consensus is an implementation of the fast consensus clustering procedure laid out in:

* Aditya Tandon, Aiiad Albeshri, Vijey Thayananthan, Wadee Alhalabi and Santo Fortunato: “[Fast consensus clustering in complex networks](https://arxiv.org/pdf/1902.04014.pdf)”, **Phys. Rev. E.**; 2019

If you use the script please cite this paper.

The procedure generates *median* or *consensus* partitions from multiple runs of a community detection algorithm. Tests on artificial benchmarks show that consensus partitions are more accurate than the ones obtained by the direct application of the clustering algorithm.

This is the refactored version of the Fast Consensus. The I/O formats are extended to be natively applicable for the [PyCaBeM](https://github.com/eXascaleInfolab/PyCABeM) clustering benchmark: arbitrary ranges of ids are supported in the input edgefile, weighted networks are supported, output directory and the number of worker processes are parameterized.  
Extended by Artem Lutov <artem@exascale.info>

## Requirements

The script requires the following:

1. [Python 3.x](https://www.python.org/downloads/)
2. [Numpy](http://www.numpy.org/)
3. [Networkx](https://networkx.github.io/)
4. [python-igraph](https://igraph.org/python/)
5. [python-louvain](https://github.com/taynaud/python-louvain)

To install the requirements:

```sh
$ pip install -r requirements.txt
```

## Usage


```
$ python3 fast_consensus.py [-h] -f INPFILE [-a ALG] [-p PARTS]
                         [--outp-parts OUTP_PARTS] [-t TAU] [-d DELTA]
                         [-w PROCS] [-o OUTDIR]

Fast consensus clustering algorithm.

optional arguments:
  -h, --help            show this help message and exit
  -f INPFILE, --network-file INPFILE
                        file with edgelist (default: None)
  -a ALG, --algorithm ALG
                        underlying clustering algorithm: louvain, lpm, cnm,
                        infomap. Note: CNM is slow (default: louvain)
  -p PARTS, --partitions PARTS
                        number of input partitions for the algorithm (default:
                        10)
  --outp-parts OUTP_PARTS
                        number of partitions to be outputted, <= input
                        partitions (default: 1)
  -t TAU, --tau TAU     used for filtering weak edges (default: None)
  -d DELTA, --delta DELTA
                        convergence parameter. Converges when less than delta
                        proportion of the edges are with wt = 1 (default:
                        0.02)
  -w PROCS, --worker-procs PROCS
                        number of parallel worker processes for the
                        clustering, it is automatically decreased to
                        min(input_partitions, cpu_num) (default: 10)
  -o OUTDIR, --output-dir OUTDIR
                        output directory (default: out_partitions)
```
Arguments description:
```
-f filename.txt
```
(Required) where `filename.txt` is an edgelist of the network.

The file can be of the form
```
0 1 0.5
0 4 1
1 3 0.3
.
.
.
```

where the first two numbers in each row are connected nodes and the third number is the edge weight. If only two numbers are provided the graph is treated as unweighted.

```
-a algorithm
```
(Optional) Here `algorithm` is the community detection method used on the network and it can be one of `louvain` ([Louvain algorithm](https://arxiv.org/abs/0803.0476)), `cnm` ([Fast greedy modularity maximization](https://arxiv.org/abs/cond-mat/0408187)), `lpm` ([Label Propagation Method](https://arxiv.org/abs/0709.2938)), `infomap` ([Infomap](http://www.mapequation.org/code.html)). If no algorithm is provided the script uses `louvain` for this purpose as the most scalable option.

```
-p partitions
```
(Optional) `p` is the number of partitions created by repeated application of the community detection algorithm. The larger this value, the more robust results are. The computational time is increased linearly with `p`.
> The number of worker process is recommended to be set to the number of input partitions.

```
--outp-parts output_partitions
```
Optional number of the output partitions (up to the number of input partitions). Note that all the partitions have absolutely equivalent clusters in case of the perfect consensus, and differ only marginally otherwise.

```
-t tau
```
(Optional) `tau` is a float between `0` and `1`. Elements of the consensus matrix with weight less than `tau` are set to zero in each step of the algorithm. If no value is provided, the code uses the value for which the chosen clustering algorithm gives the best performance on the [LFR benchmark graph](https://arxiv.org/abs/0805.4770)

```
-d delta
```
(Optional) `delta` should be a float between `0.02` and `0.1`. The procedure ends when less than `delta` fraction of the edges have a weight not equal to 1. The lower delta, the more time is required for the algorithm convergence given a fixed `tau`.


#### Example Usage

```
$ python fast_consensus.py -f examples/karate_club.txt -o res_louv -a louvain -p 50 -t 0.2 -d 0.1
```

The file `examples/karate_club.txt` is provided.


## Output
A folder `res_louv` is created with `--outp-parts` different files. Each file represents a partition; each line in the file lists all nodes belonging to a community.

For example, a run with `--outp-parts = 2` will create two files `karate_club_d0.1_0.cnl` and `karate_club_d0.1_1.cnl`. Each file will be in the form:
```
0 1 2 5 7 8 9
3 4 6 10 11
```
This represents a partition with two communities : `{0, 1, 2, 5, 7, 8, 9}` and `{3, 4, 6, 10, 11}`

## Related Projects
- [xmeasures](https://github.com/eXascaleInfolab/xmeasures)  - Extrinsic quality (accuracy) measures evaluation for the overlapping clustering on large datasets: family of mean F1-Score (including clusters labeling), Omega Index (fuzzy version of the Adjusted Rand Index) and standard NMI (for non-overlapping clusters).
- [GenConvNMI](https://github.com/eXascaleInfolab/GenConvNMI) - Overlapping NMI evaluation that is compatible with the original NMI (unlike the `onmi`).
- [OvpNMI](https://github.com/eXascaleInfolab/OvpNMI) - Another method of the NMI evaluation for the overlapping clusters (communities) that is not compatible with the standard NMI value unlike GenConvNMI, but it is much faster and yields exact results unlike probabilistic results with some variance in GenConvNMI.
- [Clubmark](https://github.com/eXascaleInfolab/clubmark) - A parallel isolation framework for benchmarking and profiling clustering (community detection) algorithms considering overlaps (covers).
- [ExecTime](https://bitbucket.org/lumais/exectime/)  - A lightweight resource consumption (RSS RAM, CPU, etc.) profiler.
