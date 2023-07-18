`ripe` lets you use the statistical programming language R in a unix pipleine.

# Examples

`ripe` is useful for one-offs and exploratory analysis. Suppose you want to fetch US COVID-19 regional wastewater concentration data, then manipulate it into a simple table for presentation. You could do this:

```
$ curl https://github.com/biobotanalytics/covid19-wastewater-data/raw/master/wastewater_by_region.csv | ripe > outfn.csv
```

curl will fetch the data, save it to a tempfile, and start R. Then in the R session you can play around, possibly coming up with:

```
> i %>% 
  select(sampling_week, average=effective_concentration_rolling_average, region) %>% 
  pivot_wider(names_from=region, values_from=average) %>% 
  write_csv()
```

Grab data from a webpage or document and drop it into R.

```
$ xclip -o -sel c | ripe
```

One of `ripe`'s forebears is [`vipe`](https://github.com/madx/moreutils/tree/master), which lets you insert an interactive text editor into a pipe. ripe and vipe go well together:

```
$ xclip -o -sel c | vipe | ripe
```

You could use the editor to remove extraneous rows, for example, then drop
it into R.

# Manual

```
USAGE:
    ripe [EXPR]
    ripe (-h | --help)
    ripe --version

OPTIONS:
    -h --help   Show this help message
    EXPR        R expression to insert in non-interactive mode
    --version   Show version

DESCRIPTION:
    ripe allows you to use R in a unix pipeline.

    ripe saves to a named tempfile whatever data it receives via stdin. 

    If EXPR is not given, ripe starts an interactive R session and assigns the
    R variable `tmpfn` to hold the name of the tempfile. The contents of the
    tempfile are parsed using readr::read_delim and assigned to a variable
    called `i` in an interactive R session. When you exit R, the contents of
    the file at `tmpfn` are written to ripe's stdout.

    The point of this is that you can use the interactive session to inspect or
    modify the data that is passing through the pipeline. For convenience, the
    `file` argument of `write_csv()` and `write_tsv()` is set to default to
    `tmpfn`.

    If EXPR is provided, ripe uses a non-interactive R session to execute

    > i %>% EXPR %>% readr::write_csv(file=tmpfn)
```

# Configuration

`ripe` is a python script. It depends on the `docopt` module for parsing command-line options.

`ripe` controls R using a custom Rprofile file. This R code is stored as a string in `ripe` and can be modified there. At runtime, ripe saves the string to a second tempfile, and instructs R to use it as an Rprofile by setting `R_PROFILE` in the user's environment.

