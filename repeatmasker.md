Pull the image and note its digest:
```
docker pull dfam/tetools:latest
docker inspect --format='{{index .RepoDigests 0}}' dfam/tetools:latest
```

Downloading human databases:
```
cd /media/ds1821p_III/aserikzhan/repeats/dfam/4.0
URL=https://www.dfam.org/releases/current/families/FamDB
for f in dfam40.0.h5.gz dfam40.curated.consensus.0.h5.gz dfam40.curated.hmm.0.h5.gz dfam40.curated.hmm.1.h5.gz; do
  wget -c $URL/$f $URL/$f.md5
done
wget $URL/README.txt

# verify each download against its checksum
for f in *.h5.gz; do
  echo "$f: expected $(cut -d' ' -f1 $f.md5)  got $(md5sum $f | cut -d' ' -f1)"
done

gunzip *.h5.gz
ls -lh
```

