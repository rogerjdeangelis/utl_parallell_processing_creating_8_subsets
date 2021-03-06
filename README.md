# utl_parallell_processing_creating_8_subsets
Running 8 processes in parallel to subset a large dataset. Not a good example but provides the algorithm.
    Macros and Parallel processing

    github
    https://github.com/rogerjdeangelis/utl_parallell_processing_creating_8_subsets

    see
    https://goo.gl/aWhjMc
    https://communities.sas.com/t5/General-SAS-Programming/Macros-and-Parallel-processing/m-p/430028



    * T1008880 Running 8 parallel SAS processes to conditional subset a dataset into 8 pieces

     You may want to try the free SPDE SAS base product.
     May give the op some ideas.

     WORKING CODE
     ============

          %macro cutDat(yyyy_qtr);
             libname sd1 "d:/sd1";
             data sd1.out_&yyyy_qtr;
                 set sd1.have(where=(yyyy_qtr="&yyyy_qtr."));
             run;quit;
          %mend cutDat;

          systask kill sys1 sys2 sys3 sys4  sys5 sys6 sys7 sys8;
          systask command "&_s -termstmt %nrstr(%cutDat(1990_1);) -log d:\log\a1.log" taskname=sys1;
          systask command "&_s -termstmt %nrstr(%cutDat(1990_2);) -log d:\log\a2.log" taskname=sys2;
          systask command "&_s -termstmt %nrstr(%cutDat(1990_3);) -log d:\log\a3.log" taskname=sys3;
          systask command "&_s -termstmt %nrstr(%cutDat(1990_4);) -log d:\log\a4.log" taskname=sys4;
          systask command "&_s -termstmt %nrstr(%cutDat(1991_1);) -log d:\log\a5.log" taskname=sys5;
          systask command "&_s -termstmt %nrstr(%cutDat(1991_2);) -log d:\log\a6.log" taskname=sys6;
          systask command "&_s -termstmt %nrstr(%cutDat(1991_3);) -log d:\log\a87log" taskname=sys7;
          systask command "&_s -termstmt %nrstr(%cutDat(1991_4);) -log d:\log\a8.log" taskname=sys8;
          waitfor sys1 sys2 sys3 sys4  sys5 sys6 sys7 sys8;

          data sd1.combine/view=combine;  * somtimes for very big datasets it is to have data partitioned;
             set sd1.out_19:;             * so do not create physical combined dataset.;
          run;quit;


          I generate the systask commands in a datastep.
          I could not easily get call execute or dosubl to work with systask.
          I could not prevent macro resolution.

    Parallel processing for a do loop


    INPUT consisting of year and quarter to create 8 datasets
    ===================================================================


    SD1.HAVE total obs=24  |   RULES
    Data to split          |
                           |
    Obs    YYYY_QTR    VAL |    Create these datasets using YYYY_QTD as filter
                           |
      1     1990_1       2 |
      2     1990_1      57 |
      3     1990_1      39 |    SD1.OUT_1990_1
                           |
      4     1990_2      61 |
      5     1990_2      70 |
      6     1990_2      92 |    SD1.OUT_1990_2
                           |
      7     1990_3      81 |
      8     1990_3      12 |
      9     1990_3      36 |    SD1.OUT_1990_3
                           |
     10     1990_4      49 |
     11     1990_4      51 |
     12     1990_4      30 |    SD1.OUT_1990_4
                           |
     13     1991_1      80 |
     14     1991_1       3 |
     15     1991_1      70 |    SD1.OUT_1991_1
                           |
     16     1991_2      21 |
     17     1991_2      33 |
     18     1991_2      98 |    SD1.OUT_1991_2
                           |
     19     1991_3      83 |
     20     1991_3      11 |
     21     1991_3      28 |    SD1.OUT_1991_3
                           |
     22     1991_4      54 |
     23     1991_4       7 |
     24     1991_4       8 |    SD1.OUT_1991_4

    PROCESS
    =======

    * note null input macro called using "-termstmt %nrstr(%cutDat(1990_1);"

    %let _s=%sysfunc(compbl(C:\Progra~1\SASHome\SASFoundation\9.4\sas.exe -sysin c:\nul
      -sasautos c:\oto -autoexec c:\oto\Tut_Oto.sas -work d:\wrk));

    * this places the macro in my autocall library(c:/oto" so the batch command can find the macro;
    * index on yyyy_qtr will speed this up;

    data _null_;file "c:\oto\cutDat.sas" lrecl=512;input;put _infile_;putlog _infile_;
    cards4;
    %macro cutDat(yyyy_qtr);
       libname sd1 "d:/sd1";
       data sd1.out_&yyyy_qtr;
           set sd1.have(where=(yyyy_qtr="&yyyy_qtr."));
       run;quit;

    %mend cutDat;
    ;;;;
    run;quit;

    %*cutDat(1991_1); * check interatively;

    * set up
    systask kill sys1 sys2 sys3 sys4  sys5 sys6 sys7 sys8;
    systask command "&_s -termstmt %nrstr(%cutDat(1990_1);) -log d:\log\a1.log" taskname=sys1;
    systask command "&_s -termstmt %nrstr(%cutDat(1990_2);) -log d:\log\a2.log" taskname=sys2;
    systask command "&_s -termstmt %nrstr(%cutDat(1990_3);) -log d:\log\a3.log" taskname=sys3;
    systask command "&_s -termstmt %nrstr(%cutDat(1990_4);) -log d:\log\a4.log" taskname=sys4;
    systask command "&_s -termstmt %nrstr(%cutDat(1991_1);) -log d:\log\a5.log" taskname=sys5;
    systask command "&_s -termstmt %nrstr(%cutDat(1991_2);) -log d:\log\a6.log" taskname=sys6;
    systask command "&_s -termstmt %nrstr(%cutDat(1991_3);) -log d:\log\a87log" taskname=sys7;
    systask command "&_s -termstmt %nrstr(%cutDat(1991_4);) -log d:\log\a8.log" taskname=sys8;
    waitfor sys1 sys2 sys3 sys4  sys5 sys6 sys7 sys8;


    WANT
    ====

    These 8 datasets

    SD1.OUT_1990_1 total obs=3

    Obs    YYYY_QTR    VAL

     1      1990_1       2
     2      1990_1      57
     3      1990_1      39
    ...

    SD1.OUT_1991_1 total obs=3

    Obs    YYYY_QTR    VAL

     1      1991_1      80
     2      1991_1       3
     3      1991_1      70

    SD1.OUT_1991_4 total obs=3

    Obs    YYYY_QTR    VAL

     1      1991_4      54
     2      1991_4       7
     3      1991_4       8

    *                _               _       _
     _ __ ___   __ _| | _____     __| | __ _| |_ __ _
    | '_ ` _ \ / _` | |/ / _ \   / _` |/ _` | __/ _` |
    | | | | | | (_| |   <  __/  | (_| | (_| | || (_| |
    |_| |_| |_|\__,_|_|\_\___|   \__,_|\__,_|\__\__,_|

    ;

    libname sd1 "d:/sd1";

    data sd1.have(drop=yyyy qtr store);
      do yyyy=1990 to 1991;
        do qtr=1 to 4;
          yyyy_qtr=cats(put(yyyy,4.),'_',put(qtr,1.));
          do store=1 to 3;
             val=int(100*uniform(5731));
             output;
          end;
        end;
      end;
    run;quit;

    *                                  _             _       _   _
     _ __ ___   __ _ _ __  _   _  __ _| |  ___  ___ | |_   _| |_(_) ___  _ __
    | '_ ` _ \ / _` | '_ \| | | |/ _` | | / __|/ _ \| | | | | __| |/ _ \| '_ \
    | | | | | | (_| | | | | |_| | (_| | | \__ \ (_) | | |_| | |_| | (_) | | | |
    |_| |_| |_|\__,_|_| |_|\__,_|\__,_|_| |___/\___/|_|\__,_|\__|_|\___/|_| |_|

    ;


    %let _s=%sysfunc(compbl(C:\Progra~1\SASHome\SASFoundation\9.4\sas.exe -sysin c:\nul
      -sasautos c:\oto -autoexec c:\oto\Tut_Oto.sas -work d:\wrk));

    data _null_;file "c:\oto\cutDat.sas" lrecl=512;input;put _infile_;putlog _infile_;
    cards4;
    %macro cutDat(yyyy_qtr);
       libname sd1 "d:/sd1";
       data sd1.out_&yyyy_qtr;
           set sd1.have(where=(yyyy_qtr="&yyyy_qtr."));
       run;quit;

    %mend cutDat;
    ;;;;
    run;quit;

    %*cutDat(1991_1);

    systask kill sys1 sys2 sys3 sys4  sys5 sys6 sys7 sys8;
    systask command "&_s -termstmt %nrstr(%cutDat(1990_1);) -log d:\log\a1.log" taskname=sys1;
    systask command "&_s -termstmt %nrstr(%cutDat(1990_2);) -log d:\log\a2.log" taskname=sys2;
    systask command "&_s -termstmt %nrstr(%cutDat(1990_3);) -log d:\log\a3.log" taskname=sys3;
    systask command "&_s -termstmt %nrstr(%cutDat(1990_4);) -log d:\log\a4.log" taskname=sys4;
    systask command "&_s -termstmt %nrstr(%cutDat(1991_1);) -log d:\log\a5.log" taskname=sys5;
    systask command "&_s -termstmt %nrstr(%cutDat(1991_2);) -log d:\log\a6.log" taskname=sys6;
    systask command "&_s -termstmt %nrstr(%cutDat(1991_3);) -log d:\log\a87log" taskname=sys7;
    systask command "&_s -termstmt %nrstr(%cutDat(1991_4);) -log d:\log\a8.log" taskname=sys8;
    waitfor sys1 sys2 sys3 sys4  sys5 sys6 sys7 sys8;

    /*
    LOG
    MPRINT(CUTDAT):  quit;
    MLOGIC(CUTDAT):  Ending execution.
    NOTE: "sys2" is not an active task/transaction.
    NOTE: There are no active tasks/transactions.
    2465  systask kill sys1 sys2 sys3 sys4  sys5 sys6 sys7 sys8;
    2466  systask command "&_s -termstmt %nrstr(%cutDat(1990_1);) -log d:\log\a1.log" taskname=sys1;
    SYMBOLGEN:  Macro variable _S resolves to C:\Progra~1\SASHome\SASFoundation\9.4\sas.exe -sysin c:\nul
    -sasautos c:\oto -autoexec c:\oto\Tut_Oto.sas -work d:\wrk
    2467  systask command "&_s -termstmt %nrstr(%cutDat(1990_2);) -log d:\log\a2.log" taskname=sys2;
    SYMBOLGEN:  Macro variable _S resolves to C:\Progra~1\SASHome\SASFoundation\9.4\sas.exe -sysin c:\nul
    -sasautos c:\oto -autoexec c:\oto\Tut_Oto.sas -work d:\wrk
    2468  systask command "&_s -termstmt %nrstr(%cutDat(1990_3);) -log d:\log\a3.log" taskname=sys3;
    SYMBOLGEN:  Macro variable _S resolves to C:\Progra~1\SASHome\SASFoundation\9.4\sas.exe -sysin c:\nul
    -sasautos c:\oto -autoexec c:\oto\Tut_Oto.sas -work d:\wrk
    2469  systask command "&_s -termstmt %nrstr(%cutDat(1990_4);) -log d:\log\a4.log" taskname=sys4;
    SYMBOLGEN:  Macro variable _S resolves to C:\Progra~1\SASHome\SASFoundation\9.4\sas.exe -sysin c:\nul
    -sasautos c:\oto -autoexec c:\oto\Tut_Oto.sas -work d:\wrk
    2470  systask command "&_s -termstmt %nrstr(%cutDat(1991_1);) -log d:\log\a5.log" taskname=sys5;
    SYMBOLGEN:  Macro variable _S resolves to C:\Progra~1\SASHome\SASFoundation\9.4\sas.exe -sysin c:\nul
    -sasautos c:\oto -autoexec c:\oto\Tut_Oto.sas -work d:\wrk
    2471  systask command "&_s -termstmt %nrstr(%cutDat(1991_2);) -log d:\log\a6.log" taskname=sys6;
    SYMBOLGEN:  Macro variable _S resolves to C:\Progra~1\SASHome\SASFoundation\9.4\sas.exe -sysin c:\nul
    -sasautos c:\oto -autoexec c:\oto\Tut_Oto.sas -work d:\wrk
    2472  systask command "&_s -termstmt %nrstr(%cutDat(1991_3);) -log d:\log\a87log" taskname=sys7;
    SYMBOLGEN:  Macro variable _S resolves to C:\Progra~1\SASHome\SASFoundation\9.4\sas.exe -sysin c:\nul
    -sasautos c:\oto -autoexec c:\oto\Tut_Oto.sas -work d:\wrk
    2473  systask command "&_s -termstmt %nrstr(%cutDat(1991_4);) -log d:\log\a8.log" taskname=sys8;
    SYMBOLGEN:  Macro variable _S resolves to C:\Progra~1\SASHome\SASFoundation\9.4\sas.exe -sysin c:\nul
    -sasautos c:\oto -autoexec c:\oto\Tut_Oto.sas -work d:\wrk
    2474  waitfor sys1 sys2 sys3 sys4  sys5 sys6 sys7 sys8;
    NOTE: Task "sys1" produced LOG/Output.
    NOTE: Task "sys3" produced LOG/Output.
    NOTE: Task "sys7" produced LOG/Output.
    NOTE: Task "sys2" produced LOG/Output.
    NOTE: Task "sys4" produced LOG/Output.
    NOTE: Task "sys8" produced LOG/Output.
    NOTE: Task "sys5" produced LOG/Output.
    NOTE: Task "sys6" produced LOG/Output.

    BATCH LOG

    NOTE: Libref SD1 was successfully assigned as follows:
          Engine:        V9
          Physical Name: d:\sd1

    NOTE: There were 3 observations read from the data set SD1.HAVE.
          WHERE yyyy_qtr='1990_1';
    NOTE: The data set SD1.OUT_1990_1 has 3 observations and 2 variables.
    NOTE: DATA statement used (Total process time):
          real time           0.05 seconds
          cpu time            0.04 seconds



    NOTE: SAS Institute Inc., SAS Campus Drive, Cary, NC USA 27513-2414
    NOTE: The SAS System used:
          real time           2.37 seconds
          cpu time            0.98 seconds

    */

    *          _       _   _                           _
     ___  ___ | |_   _| |_(_) ___  _ __     __ _ _   _| |_ ___
    / __|/ _ \| | | | | __| |/ _ \| '_ \   / _` | | | | __/ _ \
    \__ \ (_) | | |_| | |_| | (_) | | | | | (_| | |_| | || (_) |
    |___/\___/|_|\__,_|\__|_|\___/|_| |_|  \__,_|\__,_|\__\___/

    ;

    data sd1.meta(drop=yyyy qtr);
      do yyyy=1990 to 1991;
        do qtr=1 to 4;
          yyyy_qtr=cats(put(yyyy,4.),'_',put(qtr,1.));
          output;
        end;
      end;
    run;quit;


    data _null_;

        file "d:/sas/cutDat.cmd" lrecl=700;

        length cmd $700;
        retain s "&_s";

        set sd1.meta;

        pgmBf4 = catx(' ','systask command "',s,'-termstmt');

        pgm    = cats('%nrstr(%cutDat(',yyyy_qtr,');)');

        pgmAft = cats('-log d:\log\a1.log" taskname=sys',put(_n_,1.));

        cmd=catx(' ',pgmBf4,pgm,pgmAft,';');

        put cmd;

    run;quit;

    options noxwait noxsync;
    %let tym=%sysfunc(time());
    systask kill sys1 sys2 sys3 sys4  sys5 sys6 sys7 sys8;
    %inc "d:/sas/cutDat.cmd";
    waitfor sys1 sys2 sys3 sys4  sys5 sys6 sys7 sys8;





