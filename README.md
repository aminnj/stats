### stats

Print basic summaries of numbers or strings on the command line. 
Requires Python 2.6 or greater, with no dependencies.
"Install" on a linux system with
```bash
curl -O https://raw.githubusercontent.com/aminnj/stats/main/stats && chmod u+x stats
# then copy to any place on your $PATH
```

If numbers are detected, count/mean/sum/... are printed out, along with their human readable forms. 
In order to shortcut piping through `awk '{print $6}'` and allow negative indexing,
the optional first argument to `stats` specifies the column number. A `-b` signals that numbers
are bytes, so base-2 is used instead of base-10 when printing humanized units.
```bash
$ seq 1 10000 | stats

    count:  10000 (10.0k)
    mean:   5000.5 (5.0k)
    std:    2886.89567991 (2.89k)
    sum:    50005000.0 (50.01M)
    min:    1.0 (1.0)
    max:    10000.0 (10.0k)

$ hadoop fs -du /cms/store/user/$USER/ | stats 1

    length: 26 (26.0)
    mean:   6.4025619941e+11 (640.26G)
    sigma:  1.54554065089e+12 (1.55T)
    sum:    1.66466611847e+13 (16.65T)
    min:    5207981.0 (5.21M)
    max:    5.34309043154e+12 (5.34T)

$ hadoop fs -du /cms/store/user/$USER/ | stats 1 -b

    length: 26 (26.0)
    mean:   6.4025619941e+11 (596.29 GiB)
    sigma:  1.54554065089e+12 (1.41 TiB)
    sum:    1.66466611847e+13 (15.14 TiB)
    min:    5207981.0 (4.97 MiB)
    max:    5.34309043154e+12 (4.86 TiB)

```

If strings are detected, their counts are summarized (similar to `uniq -c`) with bars.
```bash
$ ls -l ~/public_html | stats -3
╭──────┬──────────────────────╮
│ Jul  │ ██████████████ (14)  │
│ Mar  │ █████████ (9)        │
│ Feb  │ ███████ (7)          │
│ Aug  │ ██████ (6)           │
│ Sep  │ █████ (5)            │
│ Nov  │ █████ (5)            │
│ Apr  │ ████ (4)             │
│ May  │ ███ (3)              │
│ Oct  │ ███ (3)              │
│ Jan  │ ██ (2)               │
│ Jun  │ █ (1)                │
│ Dec  │ █ (1)                │
╰──────┴──────────────────────╯

# Only ascii if output is piped/redirected
$ condor_q -af MATCH_EXP_JOB_Site | stats > temp.txt ; cat temp.txt
UCSD       | **** (4)
Vanderbilt | ** (2)
undefined  | * (1)
```

### side notes

Note that column selection can be done before piping into `stats` with
a handy function like
```bash
function col {
    if [ $# -lt 1 ]; then
        echo "usage: col <column #>"
        return 1
    fi
    num=$1
    if [[ $num -lt 0 ]]; then
        awk "{print \$(NF+$((num+1)))}"
    else
        awk -v x=$num '{print $x}'
    fi
}
```

```bash
$ seq 1 100 | xargs -n 2 | col 2 | stats
```
