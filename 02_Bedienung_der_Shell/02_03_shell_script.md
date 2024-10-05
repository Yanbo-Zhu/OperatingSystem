

# 1 `$@`

`$@` refers to all of a shell script’s command-line arguments.
代表 所有 在 xx.sh 后 加入的 argument 的值 

we use the special variable `$@`, which means, ‘All of the command-line arguments to the shell script’. We also should put `$@` inside double-quotes to handle the case of arguments containing spaces (`"$@"` is special syntax and is equivalent to `"$1"` `"$2"` …).


example 1
```
$ nano sorted.sh

# Sort files by their length.
# Usage: bash sorted.sh one_or_more_filenames
wc -l "$@" | sort -n

```

```
bash sorted.sh *.pdb ../creatures/*.dat

9 methane.pdb
12 ethane.pdb
15 propane.pdb
20 cubane.pdb
21 pentane.pdb
30 octane.pdb
163 ../creatures/basilisk.dat
163 ../creatures/minotaur.dat
163 ../creatures/unicorn.dat
596 total
```

example2 
```
nano do-stats.sh

# Calculate stats for data files.
for datafile in "$@"
do
    echo $datafile
    bash goostats.sh $datafile stats-$datafile
done
```


等效于 
```
bash do-stats.sh NENE*A.txt NENE*B.txt

bash do-stats.sh NENE*A.txt NENE*B.txt | wc -l


# Calculate stats for Site A and Site B data files.
for datafile in NENE*A.txt NENE*B.txt
do
    echo $datafile
    bash goostats.sh $datafile stats-$datafile
done

```

# 2 `$1`, `$2`

`$1`, `$2`, etc., refer to the first command-line argument, the second command-line argument, etc.





