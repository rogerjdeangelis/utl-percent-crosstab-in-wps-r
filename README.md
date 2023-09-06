# utl-percent-crosstab-in-wps-r
Percent crosstab in wps r uses normalization for wps
    %let pgm=utl-percent-crosstab-in-wps-r;

    Percent crosstab in wps r uses normalization for wps

    github
    https://github.com/rogerjdeangelis/utl-percent-crosstab-in-wps-r

    This is a very common problem.

    Perhaps normailzation is a better initial structure for this data
    Many SAS procedures can summarize normalixed data, ie procs, freq, tabulate, report, means and summary.
    I have examples of all of them in my repos.

         1 wps proc corresp
         2 R
           https://stackoverflow.com/users/3358272/r2evans
         3 related repos

    https://stackoverflow.com/questions/77039266/calculate-percentage-of-responses-for-each-column-in-r

    /*                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    */
    options validvarname=upcase;
    libname sd1 "d:/sd1";

    data sd1.have(drop=user);
     informat User Q1 Q2 Q3 $18.;
     input User Q1 Q2 Q3;
    cards4;
    User_1 Never Sometimes Always
    User_2 Rarely Rarely Rarely
    User_3 Sometimes Sometimes Sometimes
    User_4 Often Often Always
    User_5 Always Rarely Rarely
    ;;;;
    run;quit;


    /**************************************************************************************************************************/
    /*                                        |                      |                                                        */
    /*   INPUT                                | Step 1 Normaize View |  Pivot and summarize column percent                    */
    /*                                        |                      |                                                        */
    /*    Q1           Q2           Q3        |   VAR    VAL         |   LABEL        Q1    Q2    Q3                          */
    /*                                        |                      |                                                        */
    /*    Never        Sometimes    Always    |   Q1     Always      |   Always       20     0    40                          */
    /*    Rarely       Rarely       Rarely    |   Q1     Never       |   Never        20     0     0                          */
    /*    Sometimes    Sometimes    Sometimes |   Q1     Often       |   Often        20    20     0                          */
    /*    Often        Often        Always    |   Q1     Rarely      |   Rarely       20    40    40                          */
    /*    Always       Rarely       Rarely    |   Q1     Sometimes   |   Sometimes    20    40    20                          */
    /*                                        |                      |                                                        */
    /*                                        |   Q2     Often       |                                                        */
    /*                                        |   Q2     Rarely      |                                                        */
    /*                                        |   Q2     Rarely      |                                                        */
    /*                                        |   Q2     Sometimes   |                                                        */
    /*                                        |   Q2     Sometimes   |                                                        */
    /*                                        |                      |                                                        */
    /*                                        |   Q3     Always      |                                                        */
    /*                                        |   Q3     Always      |                                                        */
    /*                                        |   Q3     Rarely      |                                                        */
    /*                                        |   Q3     Rarely      |                                                        */
    /*                                        |   Q3     Sometimes   |                                                        */
    /*                                        |                      |                                                        */
    /**************************************************************************************************************************/

    /*---- NORMALIZE                                                         ----*/

    proc datasets lib=sd1 nolist nodetails;delete havNrm; run;quit;

    %utl_submit_wps64x('
    libname sd1 "d:/sd1";
    data sd1.havNrm;
        length var $32;
        set sd1.have ;
        array ans q:;
        do over ans;
             var=vname(ans);
             val=ans;
             output;
        end;
     drop user q: ;
    run;quit;
    ');

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /*     VAR    VAL                                                                                                         */
    /*                                                                                                                        */
    /*     Q1     Never                                                                                                       */
    /*     Q2     Sometimes                                                                                                   */
    /*     Q3     Always                                                                                                      */
    /*     Q1     Rarely                                                                                                      */
    /*     Q2     Rarely                                                                                                      */
    /*     Q3     Rarely                                                                                                      */
    /*     Q1     Sometimes                                                                                                   */
    /*     Q2     Sometimes                                                                                                   */
    /*     Q3     Sometimes                                                                                                   */
    /*     Q1     Often                                                                                                       */
    /*     Q2     Often                                                                                                       */
    /*     Q3     Always                                                                                                      */
    /*     Q1     Always                                                                                                      */
    /*     Q2     Rarely                                                                                                      */
    /*     Q3     Rarely                                                                                                      */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    proc datasets lib=sd1 nolist nodetails;delete want; run;quit;

    %utl_submit_wps64x('
    libname sd1 "d:/sd1";
    ods output ColProfilesPct=sd1.want;
    proc corresp data=sd1.havNrm dim=1 observed all print=both ;
     tables val, var;
    run;quit;
    proc print data=sd1.want;
    run;quit;
    ');

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /* COLUMN PERCENTS                                                                                                        */
    /*                                                                                                                        */
    /*  LABEL        Q1    Q2    Q3                                                                                           */
    /*                                                                                                                        */
    /*  Always       20     0    40                                                                                           */
    /*  Never        20     0     0                                                                                           */
    /*  Often        20    20     0                                                                                           */
    /*  Rarely       20    40    40                                                                                           */
    /*  Sometimes    20    40    20                                                                                           */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*___    ____
    |___ \  |  _ \
      __) | | |_) |
     / __/  |  _ <
    |_____| |_| \_\

    */
    %utl_submit_wps64x('

    proc r;
    submit;
    library(dplyr);
    library(tidyr);
    User <-c("User_1","User_2","User_3","User_4","User_5");
    Q1 <- c("Never", "Rarely", "Sometimes", "Often", "Always");
    Q2<- c("Sometimes", "Rarely", "Sometimes", "Often", "Rarely");
    Q3 <- c("Always", "Rarely", "Sometimes", "Always", "Rarely");
    data1 <- data.frame(User, Q1, Q2, Q3);
    print(User);
    facs <- c("Never", "Rarely", "Sometimes", "Often", "Always");
    data1 %>%
      pivot_longer(cols = -User) %>%
      mutate(value = factor(value, levels = facs)) %>%
      count(name, value) %>%
      pivot_wider(
        id_cols = value, names_from = name, values_from = n,
        values_fill = 0L) %>%
      mutate(across(starts_with("Q"), ~ 100 * . / sum(.)));
    endsubmit;
    run;quit;
    ');

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /* The WPS System                                                                                                         */
    /*                                                                                                                        */
    /* [1] "User_1" "User_2" "User_3" "User_4" "User_5"                                                                       */
    /* # A tibble: 5 x 4                                                                                                      */
    /*   value        Q1    Q2    Q3                                                                                          */
    /*   <fct>     <dbl> <dbl> <dbl>                                                                                          */
    /* 1 Never        20     0     0                                                                                          */
    /* 2 Rarely       20    40    40                                                                                          */
    /* 3 Sometimes    20    40    20                                                                                          */
    /* 4 Often        20    20     0                                                                                          */
    /* 5 Always       20     0    40                                                                                          */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*____            _       _           _
    |___ /   _ __ ___| | __ _| |_ ___  __| |  _ __ ___ _ __   ___  ___
      |_ \  | `__/ _ \ |/ _` | __/ _ \/ _` | | `__/ _ \ `_ \ / _ \/ __|
     ___) | | | |  __/ | (_| | ||  __/ (_| | | | |  __/ |_) | (_) \__ \
    |____/  |_|  \___|_|\__,_|\__\___|\__,_| |_|  \___| .__/ \___/|___/
                                                      |_|
    */

    https://github.com/rogerjdeangelis/utl_sort_summarize_and_transpose_multiple_variable_and_create_output_dataset
    https://github.com/rogerjdeangelis/utl_sort_summarize_set_merge_using_functions_in_formats_groupformat_fcmp
    https://github.com/rogerjdeangelis/utl_sort_summarize_transpose_and_format_in_1_datastep
    https://github.com/rogerjdeangelis/utl_sort_transpose_and_summarize_a_dataset_using_just_one_proc_report
    https://github.com/rogerjdeangelis/utl-sort-summarize-transpose-with-minimal-code-in-one-proc
    https://github.com/rogerjdeangelis/utl-sort-transpose-and-summarize-with-output-dataset-using-just-one-proc
    https://github.com/rogerjdeangelis/utl_sort_transpose_and_summarize_in_one_proc_v2
    https://github.com/rogerjdeangelis/utl_sort_transpose_summarize
    https://github.com/rogerjdeangelis/utl-create-a-sorted-summarized-and-transposed-crosstab-dataset-using-a-single-proc
    https://github.com/rogerjdeangelis/utl-drop-down-from-sas-to-R-and-summarize-weight-by-sex
    https://github.com/rogerjdeangelis/utl-minimmum-code-to-transpose-and-summarize-a-skinny-to-fat-with-sas-wps-r-and-python
    https://github.com/rogerjdeangelis/utl-summarize-by-group-within-a-date-range-parallel-processing-systask
    https://github.com/rogerjdeangelis/utl-the-all-powerfull-proc-report-to-create-transposed-sorted-and-summarized-output-datasets
    https://github.com/rogerjdeangelis/utl_complex_logic_applied_to_groups_of_data_with_summarized_stats
    https://github.com/rogerjdeangelis/utl_select_a_sas_wps_dataset_from_windows_explorer_and_summarize_the_data

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */
