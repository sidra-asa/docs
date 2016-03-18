> Author: sidra-asa

# Multi-processing in bash, getting the file type counts 

If we want to know which types are the files and the counts,<p>
typically we use following command:

    $ file * | less

However, it will take a long time if there are many files.<p>
In this case, we can use Multi-processing to shrink processing time.<p>
Following is the example:

## Instruction

Let's pretend we have a directory named files, <p>
with hundreds of thousand files in it.<p>
Here's the procedure:

* Get all names of the files.
* Get file types of the files in parallel way.
* Sort and uniq count all types

### Get file name list

    $ for i in `ls files`; do echo $i >> file_list; done

### Get file types

Split the file_list with split tool,<p>
We split it into one hundred thousand lines per file. 
    
    $ split -l 100000 file_list

The output files are named by prefix 'x':    

    $ ls
    
    xaa xab xac xae .........

Now we parallelly get the file types of the files: 

    $ for i in `ls x*`; do (for j in `cat $i`; do file files/$j >> file_type; done & ); done

### Sort and uniq count

The result is in file_type.<p>
It still take a long time to sort and uniq the file type.<p>
We use similar way to process them parallelly.
    
    $ rm x*
    $ cut -d': ' -f2 file_type >> type
    $ split -l 100000 type
    $ for i in `ls x*`; do ( cat $i |sort|uniq -c|sort -rn >> $i\_sort & ); done

The result are in the file named by prefix 'x' and postfix '_sort'.<p>

    $ cat xaa_sort
    
    32374 HTML document, ASCII text
    11164 HTML document, ASCII text, with CRLF, LF line terminators
    2664 HTML document, UTF-8 Unicode text, with very long lines, with CRLF, LF line terminators
    2210 HTML document, ASCII text, with CRLF line terminators
    198 PE32 executable (GUI) Intel 80386, for MS Windows

We use awk tool to sum the results,<p>
meanwhile, we hope the final result should be like this:

> 640934  HTML document<p>
> 3506  PE32 executable (GUI) Intel 80386<p>
> 2321  ASCII text<p>
> 393  XML document text

We take comma as separator to get first field:

    $ cut -d',' -f 1 *sort |awk '{count=$1;$1=""; a[$0]+=count}END{for ( i in a ){print a[i], i}}'|sort -rn|less

Enjoy the results!
