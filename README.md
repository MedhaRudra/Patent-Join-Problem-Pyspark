
# Patent Join in PySpark

Task: Use PySpark's RDD and DataFrame interfaces to solve the patent join problem.

## The Patent join problem explained

The goal of the patent join problem is to find *self-state patent citations*. Given are two datasets, `cite75_99.txt.gz` and `apat63_99.txt.gz`. The `Makefile` contains rules to download those data files.

The `acite75_99.txt` file contains a citation index, of the form
```
CITING CITED
```
where both `CITING` and `CITED` are integers. Each line
indicates that patent number `CITING` cites patent `CITED`.

The `pat63_99.txt` file contains the patent number, an (optional)
state in which the patent is filed and the total number of citations
made.

Our job is to augment the data in `pat63_99.txt` to include a column
indicating the number of patents cited that originate *from the same
state*. This data can only be calculated for patents that
have originating state information (and thus, only those from the US) and only for cited patents that provide that information. 

For example, 
patent 6009554 (the last patent in pat63_99.txt) cited 9 patents. Those patents were awarded to people in
* NY, 
* IL, 
* Great Britain (no state), 
* NY, 
* NY,
* FL,
* NY,
* NY,
* NY. 

For the first part, we would update the line:

```
6009554,1999,14606,1997,"US","NY",219390,2,,714,2,22,9,0,1,,,,12.7778,0.1111,0.1111,,
```

To be: 
```
6009554,1999,14606,1997,"US","NY",219390,2,,714,2,22,9,0,1,,,,12.7778,0.1111,0.1111,,6
```

The last value `,6` is the number of same-state citations. We will
report the ten patents that have the most self-state citations sorted in descending order. These patents are shown in the table below:

![Top 10 self-state citations](top-10-same-state-patents.png)


To do this, we will first need do a "data join” of the citations and
the patent data - for each cited patent, we'll need to determine the
state of the cited patent. We can then use that information to
produce the augmented patent information.

It's useful to produce an intermediate result like

|Cited|Cited_State|Citing|Citing_State|
|-----|-----|------|-----|
|2134795	|None	|5654603	|OH
|2201699	|None	|5654603	|OH
|3031593	|None	|5654603	|OH
|3093764	|OH	|5654603	|OH
|3437858	|OH	|5654603	|OH
|3852137	|PA	|5654603	|OH
|3904724	|PA	|5654603	|OH

This table says that patent `3852137` is from `PA` and `5654603` is from `OH`.
We construct this for each cited patent. From this, it's simple to determine
how many patents are self-sited for a given patent data line and then group and count those.

There are some complications:
* Not all patents in the 'cited' table are in the 'patent' table
* Not all patents cite other patents
* Not all patents are cited by other patents
* Lastly, the NaN/Null value used by PySpark makes sorting values involving Nan/Null and numeric values problematic

## Output
- [RDD Output](top-subsample-rdd.png)
- [Dataframe Output](top-10-same-state-patents.png)
