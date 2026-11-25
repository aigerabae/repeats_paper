# dustmasker

**Category:** low-complexity masker
**What it does:** finds low-complexity DNA (e.g., simple repeats, poly-A runs) with the symmetric DUST algorithm, which scores 3-mer composition in a sliding window. It does not classify repeats.
**Reference:** Morgulis A, et al. (2006). A fast and symmetric DUST implementation to mask low-complexity DNA sequences. *J Comput Biol* 13, 1028–1040. https://doi.org/10.1089/cmb.2006.13.1028

## Version

| Item | Value |
|---|---|
| Tool version | dustmasker 1.0.0 (BLAST+ 2.17.0, build 2025-07-29) |
| Container | `ncbi/blast@sha256:81f118d2e4f7e11494d27fdbb99c9430423105afff50c4ae158db41d58a3fc57` |
| Pulled | 2026-11-25 |
| Threads | 1 (dustmasker is single-threaded) |
| Parameters | defaults |

## Prerequisites (once per server)

```bash
BASE=/media/ds1821p_III/aserikzhan/repeats
IMAGE=ncbi/blast@sha256:81f118d2e4f7e11494d27fdbb99c9430423105afff50c4ae158db41d58a3fc57

docker pull $IMAGE

# GNU time is mounted into the container to measure the tool's own runtime and memory.
# Copied into the project folder because the (snap) Docker engine cannot see /usr/bin on the host.
mkdir -p $BASE/bin && cp /usr/bin/time $BASE/bin/time
```

## Run (T2T-CHM13)

```bash
GENOME=chm13
FASTA=/data/genomes/chm13/chm13v2.0.fa
OUT=runs/dustmasker/$GENOME
mkdir -p $BASE/$OUT

docker run --rm \
  --cpuset-cpus=0-15 --cpuset-mems=0-3 --memory=56g --memory-swap=56g \
  -v $BASE:/data -v $BASE/bin/time:/usr/bin/time:ro \
  $IMAGE \
  /usr/bin/time -v -o /data/$OUT/time.log \
  dustmasker -in $FASTA -outfmt interval \
             -out /data/$OUT/$GENOME.dust.intervals
```

Standard allocation: 16 CPUs (0–15, NUMA nodes 0–3), 56 GB RAM, no swap. For other genomes, change `GENOME` and `FASTA`.

## Outputs

| File | Content |
|---|---|
| `<genome>.dust.intervals` | One `>chrN` header per sequence, then one masked interval per line (`start - end`) |
| `time.log` | GNU time report: wall time, CPU time, peak memory (`Maximum resident set size`), exit status |

Before converting intervals to BED for evaluation, check dustmasker's coordinate convention (0- or 1-based, inclusive end) against a few known low-complexity regions.

## Checks after a run

```bash
cd $BASE/$OUT
grep -E "Exit status|Elapsed \(wall|Maximum resident" time.log   # exit status must be 0
grep -c "^>" $GENOME.dust.intervals                               # CHM13: 24
head -3 $GENOME.dust.intervals
```

## Run log

| Genome | Date | Wall time | Peak memory | Exit | Server load (start/end) | Notes |
|---|---|---|---|---|---|---|
| CHM13 | 2026-11-25 | | | | | |

## Notes

- First attempt (2026-11-25) failed with exit 126: `/usr/bin/time` outside the container could not start `docker` (server has both regular and snap Docker installed). Fixed by timing inside the container. Timing from outside would also have measured the Docker client's memory, not dustmasker's.
- dustmasker uses 1 CPU, so the 16-CPU allocation mostly sits idle; report CPU time alongside wall time.
