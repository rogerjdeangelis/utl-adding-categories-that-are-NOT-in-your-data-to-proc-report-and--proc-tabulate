# utl-adding-categories-that-are-NOT-in-your-data-to-proc-report-and--proc-tabulate
Adding-categories-that-are-NOT-in-your-data-to-proc-report-and--proc-tabulate
    Adding-categories-that-are-NOT-in-your-data-to-proc-report-and--proc-tabulate

    The op wanted the sum for the missing category to be zero.

    github
    https://tinyurl.com/yyvdq64w
    https://github.com/rogerjdeangelis/utl-adding-categories-that-are-NOT-in-your-data-to-proc-report-and--proc-tabulate

    I don't recommend all of these solutions;
    I experiment with dosubl even when it may not be the best solution.
    Experimentation leads to discovery.

    There is a 'BUG' in both report and tabulate, the 'ods output' table
    does not have '0' for the missing category or the mis2zro format.
    The 'out=' datasets have '.' , which is correct.

    Three Solutions  ( 'ods output' does not honor the listing output value=. not 0 )

         1. proc tabulate with misstext='0'
         2. proc report using two formats  (does not support misstext=0)
         3. proc report using option missing="0" (does not support middtext=0)


    https://communities.sas.com/t5/New-SAS-User/Add-Record/m-p/554026

    BallardW
    https://communities.sas.com/t5/user/viewprofilepage/user-id/13884

    *_                   _
    (_)_ __  _ __  _   _| |_
    | | '_ \| '_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    ;

    data have;
       input source $ value;
       format value 1.;
    cards4;
    A  1
    A -1
    B 1
    B 1
    B 1
    ;;;;
    run;quit;

    *           _
     _ __ _   _| | ___  ___
    | '__| | | | |/ _ \/ __|
    | |  | |_| | |  __/\__ \
    |_|   \__,_|_|\___||___/

    ;

     WORK.HAVE total obs=5

                         |  RULES
      SOURCE    VALUE    |  SOURCE    SUM
                         |
        A          1     |
        A         -1     |   A         0 (1 - 1)
                         |
        B          1     |
        B          1     |
        B          1     |   B         3 (1+1+1)
                         |
                         |   C         0 (add even though not in data)

    *            _              _
      ___  _   _| |_ _ __  _   _| |_
     / _ \| | | | __| '_ \| | | | __|
    | (_) | |_| | |_| |_) | |_| | |_
     \___/ \__,_|\__| .__/ \__,_|\__|
                    |_|
    ;


    *******************************************
    PROC TABULATE LISTING  and output dataset *
    *******************************************

    ------------------------
    |            |   VALUE |
    |------------+---------|
    |SOURCE      |         |
    |------------|         |
    |A           |    0    |
    |------------+---------|
    |B           |    3    |
    |------------+---------|
    |C           |    0    |
    ------------------------


    WORK.TABOUT total obs=3

               VALUE_
     SOURCE      SUM

       A          0
       B          3
       C          .     (ods output has '.' but should have '0'?)
                        (ok that the out=wanr has '.')

    ***************************************
    PROC REPORT LISTING AND OUTPUT DATASET*
    ***************************************

     SOURCE     SUM
     ---------------
       A          0
       B          3
       C          0

    AND A SAS DATASET

    WORK.WANT total obs=3

      SOURCE    VALUE

        A         0
        B         3
        C         ,      (ods output has '.' but should have '0'?)
                         (ok that the out=wanr has '.')

    *          _       _   _
     ___  ___ | |_   _| |_(_) ___  _ __  ___
    / __|/ _ \| | | | | __| |/ _ \| '_ \/ __|
    \__ \ (_) | | |_| | |_| | (_) | | | \__ \
    |___/\___/|_|\__,_|\__|_|\___/|_| |_|___/

    ;

    ************************************
    1. proc tabulate with misstext='0' *
    ************************************

    * just in case they exist;
    proc catalog cat=work.formats;
      delete source /et=formatc;
    run;quit;
    options missing=".";


    proc tabulate data=have
      (where=( 0 = %sysfunc(
        dosubl('
           proc format;
              value $source
              "A" = "A"
              "B" = "B"
              "C" = "C";
           run;quit;
        ')))) missing out=tabout (Drop=_:);;
       class source /preloadfmt  ;
       format source $source.;
       var value;
       table source,
             value*sum=' ' * f=6.
             / printmiss misstext="0" rts=10 ;
      ;
    run;quit;


    ******************************************************************
    2. proc report using two formats  (does not support misstext=0)  *
    ******************************************************************

    * just in case they exist;
    proc catalog cat=work.formats;
      delete source /et=formatc;
      delete mis2zro /et=format;
    run;quit;
    options missing=".";


    proc report data=have
      (where=( 0 = %sysfunc(
        dosubl('
           proc format;
              value $source
              "A" = "A"
              "B" = "B"
              "C" = "C";
              value mis2zro
              .= 0;
           run;quit;
        ')))) out=want (drop=_break_:) missing completerows headline;
    cols source value;
    define source /group width=7 format=$source. preloadfmt;
    define value /sum width=4 format=mis2zro. "SUM";
    run;quit;


    ************************************************************************
    3. proc report using option missing="0" (does not support middtext=0)  *
    ************************************************************************

    proc catalog cat=work.formats;
      delete source /et=formatc;
    run;quit;
    options missing=".";


    options missing="0";
    proc report data=have (where=( 0 = %sysfunc(
        dosubl('
           proc format;
             value $source
              "A" = "A"
              "B" = "B"
              "C" = "C";
           run;quit;
        ')))) missing completerows headline  out=want;
    cols source value;
    define source /group width=7 format=$source. preloadfmt;
    define value /sum  "SUM" width=5;
    run;quit;
    options missing=".";




