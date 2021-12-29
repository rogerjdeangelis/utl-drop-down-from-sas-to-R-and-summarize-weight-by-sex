# utl-drop-down-from-sas-to-R-and-summarize-weight-by-sex
Summarize weights in sashelp.class using SAS nad R integration 
    %let pgm=utl-drop-down-from-sas-to-R-and-summarize-weight-by-sex;

    Summarize weights in sashelp.class using SAS nad R integration

    github
    https://tinyurl.com/36fd6d2n
    https://github.com/rogerjdeangelis/utl-drop-down-from-sas-to-R-and-summarize-weight-by-sex


    gitHub related
    https://github.com/rogerjdeangelis?tab=repositories&q=+drop+down&type=&language=&sort=

    Macro on the end of this message and in gitHub


    PROCESS

       Create class sas dataset on local drive
       Create SAS macro variable to pass to R
       Drop down from SAS to R  (have drop downs to PERL, PYTHON)
       Pass variable name to R using a SAS macro variable
       Pass SAS class table to R dataframe
       Summarize the variable by sex
       Create a macro variable in R and pass the new macro variable back to SAS
       Export the summarived data back to SAS
       Return to SAS nand process R outputs

    /*
     _                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    */

    * copy sashelp.class to local drive;
    libname sd1 "c:/sd1";
    options validvarname=upcase;

    data sd1.class;
       set sashelp.class;
    run;quit;

    /*
    MACRO VARIABLE

    %let var=WEIGHT;

    SAS TABLE

    Up to 40 obs SD1.CLASS total obs=19 22DEC2021:07:45:49

    Obs    NAME       SEX    AGE    HEIGHT    WEIGHT
                                      xpy
      1    Alfred      M      14     69.0      112.5
      2    Alice       F      13     56.5       84.0
      3    Barbara     F      13     65.3       98.0
      4    Carol       F      14     62.8      102.5
      5    Henry       M      14     63.5      102.5
      6    James       M      12     57.3       83.0
      7    Jane        F      12     59.8       84.5
      8    Janet       F      15     62.5      112.5
      9    Jeffrey     M      13     62.5       84.0
     10    John        M      12     59.0       99.5
    ....
    */

    /*           _               _
      ___  _   _| |_ _ __  _   _| |_
     / _ \| | | | __| `_ \| | | | __|
    | (_) | |_| | |_| |_) | |_| | |_
     \___/ \__,_|\__| .__/ \__,_|\__|
                    |_|

    * SAS MACRO VARIABLE "STUDENTS" FROM R ;

    Total Numner of Student STUDENTS=19

    STUDENTS    SEX    SEXCNT     WGTAVG     WGTSTD    MDLWGT    MINWGT    MAXWGT

       19        F        9       90.111    19.3839     90.00     50.5      112.5
       19        M       10      108.950    22.7272    107.25     83.0      150.0

     _ __  _ __ ___   ___ ___  ___ ___
    | `_ \| `__/ _ \ / __/ _ \/ __/ __|
    | |_) | | | (_) | (_|  __/\__ \__ \
    | .__/|_|  \___/ \___\___||___/___/
    |_|
    */

    %let var=WEIGHT;

    %utlfkil(c:/xpt/stats.xpt);

    %utl_submit_r64("
      library(haven);
      library(SASxport);
      library(sqldf);
      class<-read_sas('c:/sd1/class.sas7bdat');
      class;
      stats<-sqldf('
         select
           (select count(*) from class) as students
          ,SEX
          ,count(*)     as sexCnt
          ,avg(&var)    as wgtAvg
          ,stdev(&var)  as wgtStd
          ,median(&var) as mdlWgt
          ,min(&var)    as minWgt
          ,max(&var)    as maxWgt
        from
           class
        group
           by SEX
        ;
        ');
      stats;
      stats$students;
      write.xport(stats,file='c:/xpt/stats.xpt');
      writeClipboard(as.character(paste(stats$students[1], collapse = ' ')));
    ",returnVar=students);

    %put Total Numner of Student &=students;

    libname xpt xport "c:/xpt/stats.xpt";

    proc print data=xpt.stats;
    run;quit;

    /*
     _ __ ___   __ _  ___ _ __ ___
    | `_ ` _ \ / _` |/ __| `__/ _ \
    | | | | | | (_| | (__| | | (_) |
    |_| |_| |_|\__,_|\___|_|  \___/

    */

    %macro utl_submit_r64(
          pgmx
         ,returnVar=N           /* set to Y if you want a return SAS macro variable from python */
         )/des="Semi colon separated set of R commands - drop down to R";
      * write the program to a temporary file;
      filename r_pgm "%sysfunc(pathname(work))/r_pgm.txt" lrecl=32766 recfm=v;
      data _null_;
        length pgm $32756;
        file r_pgm;
        pgm=&pgmx;
        put pgm;
        putlog pgm;
      run;
      %let __loc=%sysfunc(pathname(r_pgm));
      * pipe file through R;
      filename rut pipe "C:/PROGRA~1/R/R-4.1.2/bin/R.exe --vanilla --quiet --no-save < &__loc";
      data _null_;
        file print;
        infile rut recfm=v lrecl=32756;
        input;
        put _infile_;
        putlog _infile_;
      run;
      filename rut clear;
      filename r_pgm clear;

      * use the clipboard to create macro variable;
      %if %upcase(%substr(&returnVar.,1,1)) ne N %then %do;
        filename clp clipbrd ;
        data _null_;
         length txt $200;
         infile clp;
         input;
         putlog "macro variable &returnVar = " _infile_;
         call symputx("&returnVar.",_infile_,"G");
        run;quit;
      %end;

    %mend utl_submit_r64;

    /*
    | | ___   __ _
    | |/ _ \ / _` |
    | | (_) | (_| |
    |_|\___/ \__, |
             |___/
    */


    1726  /*
    1727   _                   _
    1728  (_)_ __  _ __  _   _| |_
    1729  | | `_ \| `_ \| | | | __|
    1730  | | | | | |_) | |_| | |_
    1731  |_|_| |_| .__/ \__,_|\__|
    1732          |_|
    1733  */
    1734  * cioy sashelp.class to local drive;
    1735  libname sd1 "c:/sd1";
    NOTE: Libref SD1 was successfully assigned as follows:
          Engine:        V9
          Physical Name: c:\sd1
    1736  options validvarname=upcase;
    1737  data sd1.class;
    1738     set sashelp.class;
    1739  run;

    NOTE: There were 19 observations read from the data set SASHELP.CLASS.
    NOTE: The data set SD1.CLASS has 19 observations and 5 variables.
    NOTE: DATA statement used (Total process time):
          real time           0.00 seconds
          user cpu time       0.00 seconds
          system cpu time     0.00 seconds
          memory              558.34k
          OS Memory           29696.00k
          Timestamp           12/22/2021 09:12:25 AM
          Step Count                        640  Switch Count  0


    1739!     quit;
    1740  /*
    1741  MACRO VARIABLE
    1742  %let var=WEIGHT;
    1743  SAS TABLE
    1744  Up to 40 obs SD1.CLASS total obs=19 22DEC2021:07:45:49
    1745  Obs    NAME       SEX    AGE    HEIGHT    WEIGHT
    1746                                    xpy
    1747    1    Alfred      M      14     69.0      112.5
    1748    2    Alice       F      13     56.5       84.0
    1749    3    Barbara     F      13     65.3       98.0
    1750    4    Carol       F      14     62.8      102.5
    1751    5    Henry       M      14     63.5      102.5
    1752    6    James       M      12     57.3       83.0
    1753    7    Jane        F      12     59.8       84.5
    1754    8    Janet       F      15     62.5      112.5
    1755    9    Jeffrey     M      13     62.5       84.0
    1756   10    John        M      12     59.0       99.5
    1757  ....
    1758  */
    1759  /*           _               _
    1760    ___  _   _| |_ _ __  _   _| |_
    1761   / _ \| | | | __| `_ \| | | | __|
    1762  | (_) | |_| | |_| |_) | |_| | |_
    1763   \___/ \__,_|\__| .__/ \__,_|\__|
    1764                  |_|
    1765  * SAS MACRO VARIABLE "STUDENTS" FROM R ;
    1766  Total Numner of Student STUDENTS=19
    1767  STUDENTS    SEX    SEXCNT     WGTAVG     WGTSTD    MDLWGT    MINWGT    MAXWGT
    1768     19        F        9       90.111    19.3839     90.00     50.5      112.5
    1769     19        M       10      108.950    22.7272    107.25     83.0      150.0
    1770   _ __  _ __ ___   ___ ___  ___ ___
    1771  | `_ \| `__/ _ \ / __/ _ \/ __/ __|
    1772  | |_) | | | (_) | (_|  __/\__ \__ \
    1773  | .__/|_|  \___/ \___\___||___/___/
    1774  |_|
    1775  */
    1776  %let var=WEIGHT;
    1777  %utlfkil(c:/xpt/stats.xpt);
    MLOGIC(UTLFKIL):  Beginning execution.
    MLOGIC(UTLFKIL):  Parameter UTLFKIL has value c:/xpt/stats.xpt
    MLOGIC(UTLFKIL):  %LOCAL  URC
    MLOGIC(UTLFKIL):  %LET (variable name is URC)
    SYMBOLGEN:  Macro variable UTLFKIL resolves to c:/xpt/stats.xpt
    SYMBOLGEN:  Macro variable URC resolves to 0
    SYMBOLGEN:  Macro variable FNAME resolves to #LN00468
    MLOGIC(UTLFKIL):  %IF condition &urc = 0 and %sysfunc(fexist(&fname)) is TRUE
    MLOGIC(UTLFKIL):  %LET (variable name is URC)
    SYMBOLGEN:  Macro variable FNAME resolves to #LN00468
    MLOGIC(UTLFKIL):  %LET (variable name is URC)
    MPRINT(UTLFKIL):   run;
    MLOGIC(UTLFKIL):  Ending execution.
    1778  %utl_submit_r64("
    MLOGIC(UTL_SUBMIT_R64):  Beginning execution.
    1779    library(haven);
    1780    library(SASxport);
    1781    library(sqldf);
    1782    class<-read_sas('c:/sd1/class.sas7bdat');
    1783    class;
    1784    stats<-sqldf('
    1785       select
    1786         (select count(*) from class) as students
    1787        ,SEX
    1788        ,count(*)     as sexCnt
    1789        ,avg(&var)    as wgtAvg
    SYMBOLGEN:  Macro variable VAR resolves to WEIGHT
    1790        ,stdev(&var)  as wgtStd
    SYMBOLGEN:  Macro variable VAR resolves to WEIGHT
    1791        ,median(&var) as mdlWgt
    SYMBOLGEN:  Macro variable VAR resolves to WEIGHT
    1792        ,min(&var)    as minWgt
    SYMBOLGEN:  Macro variable VAR resolves to WEIGHT
    1793        ,max(&var)    as maxWgt
    SYMBOLGEN:  Macro variable VAR resolves to WEIGHT
    1794      from
    1795         class
    1796      group
    1797         by SEX
    1798      ;
    1799      ');
    1800    stats;
    1801    stats$students;
    1802    write.xport(stats,file='c:/xpt/stats.xpt');
    1803    writeClipboard(as.character(paste(stats$students[1], collapse = ' ')));
    1804  ",returnVar=students);
    MLOGIC(UTL_SUBMIT_R64):  Parameter PGMX has value "  library(haven);  library(SASxport);  library(sqldf);  class<-read_sas('c:/sd1/class.sas7bdat');  class;
          stats<-sqldf('     select       (select count(*) from class) as students      ,SEX      ,count(*)     as sexCnt      ,avg(WEIGHT)    as wgtAvg      ,stdev(WEIGHT)
          as wgtStd      ,median(WEIGHT) as mdlWgt      ,min(WEIGHT)    as minWgt      ,max(WEIGHT)    as maxWgt    from       class    group       by SEX    ;    ');  stats;
           stats$students;  write.xport(stats,file='c:/xpt/stats.xpt');  writeClipboard(as.character(paste(stats$students[1], collapse = ' ')));"
    MLOGIC(UTL_SUBMIT_R64):  Parameter RETURNVAR has value students
    MPRINT(UTL_SUBMIT_R64):   * write the program to a temporary file;
    MPRINT(UTL_SUBMIT_R64):   filename r_pgm "c:\wrk\_TD27488_RDEANGELISL5410_/r_pgm.txt" lrecl=32766 recfm=v;
    MPRINT(UTL_SUBMIT_R64):   data _null_;
    MPRINT(UTL_SUBMIT_R64):   length pgm $32756;
    MPRINT(UTL_SUBMIT_R64):   file r_pgm;
    SYMBOLGEN:  Macro variable PGMX resolves to "  library(haven);  library(SASxport);  library(sqldf);  class<-read_sas('c:/sd1/class.sas7bdat');  class;  stats<-sqldf('
                select       (select count(*) from class) as students      ,SEX      ,count(*)     as sexCnt      ,avg(WEIGHT)    as wgtAvg      ,stdev(WEIGHT)  as wgtStd
                 ,median(WEIGHT) as mdlWgt      ,min(WEIGHT)    as minWgt      ,max(WEIGHT)    as maxWgt    from       class    group       by SEX    ;    ');  stats;
                stats$students;  write.xport(stats,file='c:/xpt/stats.xpt');  writeClipboard(as.character(paste(stats$students[1], collapse = ' ')));"
    MPRINT(UTL_SUBMIT_R64):   pgm="  library(haven);  library(SASxport);  library(sqldf);  class<-read_sas('c:/sd1/class.sas7bdat');  class;  stats<-sqldf('     select
    (select count(*) from class) as students      ,SEX      ,count(*)     as sexCnt      ,avg(WEIGHT)    as wgtAvg      ,stdev(WEIGHT)  as wgtStd      ,median(WEIGHT) as
    mdlWgt      ,min(WEIGHT)    as minWgt      ,max(WEIGHT)    as maxWgt    from       class    group       by SEX    ;    ');  stats;  stats$students;
    write.xport(stats,file='c:/xpt/stats.xpt');  writeClipboard(as.character(paste(stats$students[1], collapse = ' ')));";
    MPRINT(UTL_SUBMIT_R64):   put pgm;
    MPRINT(UTL_SUBMIT_R64):   putlog pgm;
    MPRINT(UTL_SUBMIT_R64):   run;

    NOTE: The file R_PGM is:
          Filename=c:\wrk\_TD27488_RDEANGELISL5410_\r_pgm.txt,
          RECFM=V,LRECL=32766,File Size (bytes)=0,
          Last Modified=22Dec2021:09:12:25,
          Create Time=22Dec2021:08:02:19

    library(haven);  library(SASxport);  library(sqldf);  class<-read_sas('c:/sd1/class.sas7bdat');  class;  stats<-sqldf('     select       (select count(*) from class) as st
    udents      ,SEX      ,count(*)     as sexCnt      ,avg(WEIGHT)    as wgtAvg      ,stdev(WEIGHT)  as wgtStd      ,median(WEIGHT) as mdlWgt      ,min(WEIGHT)    as minWgt
        ,max(WEIGHT)    as maxWgt    from       class    group       by SEX    ;    ');  stats;  stats$students;  write.xport(stats,file='c:/xpt/stats.xpt');  writeClipboard(a
    s.character(paste(stats$students[1], collapse = ' ')));
    NOTE: 1 record was written to the file R_PGM.
          The minimum record length was 568.
          The maximum record length was 568.
    NOTE: DATA statement used (Total process time):
          real time           0.01 seconds
          user cpu time       0.00 seconds
          system cpu time     0.00 seconds
          memory              375.93k
          OS Memory           29696.00k
          Timestamp           12/22/2021 09:12:25 AM
          Step Count                        641  Switch Count  0


    MLOGIC(UTL_SUBMIT_R64):  %LET (variable name is __LOC)
    MPRINT(UTL_SUBMIT_R64):   * pipe file through R;
    SYMBOLGEN:  Macro variable __LOC resolves to c:\wrk\_TD27488_RDEANGELISL5410_\r_pgm.txt
    MPRINT(UTL_SUBMIT_R64):   filename rut pipe "C:/PROGRA~1/R/R-4.1.2/bin/R.exe --vanilla --quiet --no-save < c:\wrk\_TD27488_RDEANGELISL5410_\r_pgm.txt";
    MPRINT(UTL_SUBMIT_R64):   data _null_;
    MPRINT(UTL_SUBMIT_R64):   file print;
    MPRINT(UTL_SUBMIT_R64):   infile rut recfm=v lrecl=32756;
    MPRINT(UTL_SUBMIT_R64):   input;
    MPRINT(UTL_SUBMIT_R64):   put _infile_;
    MPRINT(UTL_SUBMIT_R64):   putlog _infile_;
    MPRINT(UTL_SUBMIT_R64):   run;

    NOTE: The infile RUT is:
          Unnamed Pipe Access Device,
          PROCESS=C:/PROGRA~1/R/R-4.1.2/bin/R.exe --vanilla --quiet --no-save < c:\wrk\_TD27488_RDEANGELISL5410_\r_pgm.txt,
          RECFM=V,LRECL=32756

    > library(haven);  library(SASxport);  library(sqldf);  class<-read_sas('c:/sd1/class.sas7bdat');  class;  stats<-sqldf('     select       (select count(*) from class) as
    students      ,SEX      ,count(*)     as sexCnt      ,avg(WEIGHT)    as wgtAvg      ,stdev(WEIGHT)  as wgtStd      ,median(WEIGHT) as mdlWgt      ,min(WEIGHT)    as minWgt
          ,max(WEIGHT)    as maxWgt    from       class    group       by SEX    ;    ');  stats;  stats$students;  write.xport(stats,file='c:/xpt/stats.xpt');  writeClipboard
    (as.character(paste(stats$students[1], collapse = ' ')));
    # A tibble: 19 x 5
       NAME    SEX     AGE HEIGHT WEIGHT
       <chr>   <chr> <dbl>  <dbl>  <dbl>
     1 Alfred  M        14   69    112.
     2 Alice   F        13   56.5   84
     3 Barbara F        13   65.3   98
     4 Carol   F        14   62.8  102.
     5 Henry   M        14   63.5  102.
     6 James   M        12   57.3   83
     7 Jane    F        12   59.8   84.5
     8 Janet   F        15   62.5  112.
     9 Jeffrey M        13   62.5   84
    10 John    M        12   59     99.5
    11 Joyce   F        11   51.3   50.5
    12 Judy    F        14   64.3   90
    13 Louise  F        12   56.3   77
    14 Mary    F        15   66.5  112
    15 Philip  M        16   72    150
    16 Robert  M        12   64.8  128
    17 Ronald  M        15   67    133
    18 Thomas  M        11   57.5   85
    19 William M        15   66.5  112
      students SEX sexCnt    wgtAvg   wgtStd mdlWgt minWgt maxWgt
    1       19   F      9  90.11111 19.38391  90.00   50.5  112.5
    2       19   M     10 108.95000 22.72719 107.25   83.0  150.0
    [1] 19 19
    >
    NOTE: 31 lines were written to file PRINT.
    Stderr output:
    Loading required package: gsubfn
    Loading required package: proto
    Loading required package: RSQLite
    NOTE: 28 records were read from the infile RUT.
          The minimum record length was 2.
          The maximum record length was 570.
    NOTE: DATA statement used (Total process time):
          real time           2.15 seconds
          user cpu time       0.01 seconds
          system cpu time     0.06 seconds
          memory              317.00k
          OS Memory           29696.00k
          Timestamp           12/22/2021 09:12:27 AM
          Step Count                        642  Switch Count  0


    MPRINT(UTL_SUBMIT_R64):   filename rut clear;
    NOTE: Fileref RUT has been deassigned.
    MPRINT(UTL_SUBMIT_R64):   filename r_pgm clear;
    NOTE: Fileref R_PGM has been deassigned.
    MPRINT(UTL_SUBMIT_R64):   * use the clipboard to create macro variable;
    SYMBOLGEN:  Macro variable RETURNVAR resolves to students
    MLOGIC(UTL_SUBMIT_R64):  %IF condition %upcase(%substr(&returnVar.,1,1)) ne N is TRUE
    MPRINT(UTL_SUBMIT_R64):   filename clp clipbrd ;
    MPRINT(UTL_SUBMIT_R64):   data _null_;
    MPRINT(UTL_SUBMIT_R64):   length txt $200;
    MPRINT(UTL_SUBMIT_R64):   infile clp;
    MPRINT(UTL_SUBMIT_R64):   input;
    SYMBOLGEN:  Macro variable RETURNVAR resolves to students
    MPRINT(UTL_SUBMIT_R64):   putlog "macro variable students = " _infile_;
    SYMBOLGEN:  Macro variable RETURNVAR resolves to students
    MPRINT(UTL_SUBMIT_R64):   call symputx("students",_infile_,"G");
    MPRINT(UTL_SUBMIT_R64):   run;

    NOTE: Variable TXT is uninitialized.
    NOTE: The infile CLP is:
          (no system-specific pathname available),
          (no system-specific file attributes available)

    macro variable students = 19
    NOTE: 1 record was read from the infile CLP.
          The minimum record length was 2.
          The maximum record length was 2.
    NOTE: DATA statement used (Total process time):
          real time           0.00 seconds
          user cpu time       0.00 seconds
          system cpu time     0.00 seconds
          memory              332.65k
          OS Memory           29696.00k
          Timestamp           12/22/2021 09:12:27 AM
          Step Count                        643  Switch Count  0


    MPRINT(UTL_SUBMIT_R64):  quit;
    MLOGIC(UTL_SUBMIT_R64):  Ending execution.
    SYMBOLGEN:  Macro variable STUDENTS resolves to 19
    1805  %put Total Numner of Student &=students;
    Total Numner of Student STUDENTS=19
    1806  libname xpt xport "c:/xpt/stats.xpt";
    NOTE: Libref XPT was successfully assigned as follows:
          Engine:        XPORT
          Physical Name: c:\xpt\stats.xpt
    1807  proc print data=xpt.stats;
    1808  run;

    NOTE: Access by observation number not available. Observation numbers will be counted by PROC PRINT.
    NOTE: There were 2 observations read from the data set XPT.STATS.
    NOTE: PROCEDURE PRINT used (Total process time):
          real time           0.00 seconds
          user cpu time       0.00 seconds
          system cpu time     0.00 seconds
          memory              342.50k
          OS Memory           29696.00k
          Timestamp           12/22/2021 09:12:27 AM
          Step Count                        644  Switch Count  0

    1808!     quit;


    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */
