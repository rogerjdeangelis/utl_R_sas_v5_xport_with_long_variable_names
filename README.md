# utl_R_sas_v5_xport_with_long_variable_names
R script that creats V5 SAS transport file that accommodates long variable names   I use the SAS label in the V5 SAS transport file to rename the 8 char variable names   Just two significant limitation left to fix with the V5 transport format       1. Slight loss with very large numbers due to IBM 8 bit exponent (IEE 754 float uses 11 bits)      2. Character length limited to 200 bytes
    R script that creates a v5 SAS transport file that accommodates long variable names

     I use the SAS label in the V5 SAS transport file to rename the 8 char variable names

     Just two significant limitation left to fix with the V5 transport format

         1. Slight loss with very large numbers due to IBM 8 bit exponent (IEE 754 float uses 11 bits)
         2. Character length limited to 200 bytes


    INPUT
    =====

       iris dataframe

          Sepal.Length Sepal.Width Petal.Length Petal.Width Species

        1          5.1         3.5          1.4         0.2  setosa
        2          4.9         3.0          1.4         0.2  setosa
        3          4.7         3.2          1.3         0.2  setosa
        4          4.6         3.1          1.5         0.2  setosa
        5          5.0         3.6          1.4         0.2  setosa
        6          5.4         3.9          1.7         0.4  setosa


    WORKING CODE
    ============

       The macro does not cycle through all the obsevations in IRIS, it
       just manipulates meta data

       libname xpt xport "d:/xpt/rxpt.xpt";
       data iris_long_names;

         %utl_rens(xpt.iris); * only manipulates meta data;
         set iris;            * iris is a SQL view that renames the xport variables;

       run;quit;

    OUTPUT   ( variable names are 11 chars suppots up to 32 characters)
    ======

       WORK.IRIS_LONGNAMES total obs=150

       Obs    SEPALLENGTH    SEPALWIDTH    PETALLENGTH    PETALWIDTH    SPECIES

         1        5.1            3.5           1.4            0.2       setosa
         2        4.9            3.0           1.4            0.2       setosa
         3        4.7            3.2           1.3            0.2       setosa
         4        4.6            3.1           1.5            0.2       setosa
         5        5.0            3.6           1.4            0.2       setosa
         6        5.4            3.9           1.7            0.4       setosa

    *                _                ____                          _
     _ __ ___   __ _| | _____  __   _| ___|  __  ___ __   ___  _ __| |_
    | '_ ` _ \ / _` | |/ / _ \ \ \ / /___ \  \ \/ / '_ \ / _ \| '__| __|
    | | | | | | (_| |   <  __/  \ V / ___) |  >  <| |_) | (_) | |  | |_
    |_| |_| |_|\__,_|_|\_\___|   \_/ |____/  /_/\_\ .__/ \___/|_|   \__|
                                                  |_|
    ;

    %utlfkil(d:/xpt/rxpt.xpt);
    %symdel rens / nowarn;

    %utl_submit_r64('
    source("c:/Program Files/R/R-3.3.2/etc/Rprofile.site",echo=T);
    library(SASxport);
    library(Hmisc);
    iris$Species<-as.character(iris$Species);
    head(iris);
    colnames(iris)<-c("SLENGTH","SWIDTH","PLENGTH","PWIDTH","SPECIES");
    label(iris$SLENGTH) <-"SEPALLENGTH";
    label(iris$SWIDTH ) <-"SEPALWIDTH";
    label(iris$PLENGTH) <-"PETALLENGTH";
    label(iris$PWIDTH ) <-"PETALWIDTH";
    label(iris$SPECIES ) <-"SPECIES";
    write.xport(iris,file="d:/xpt/rxpt.xpt");
    ');

    *          _       _   _
     ___  ___ | |_   _| |_(_) ___  _ __
    / __|/ _ \| | | | | __| |/ _ \| '_ \
    \__ \ (_) | | |_| | |_| | (_) | | | |
    |___/\___/|_|\__,_|\__|_|\___/|_| |_|
    ;

    %macro utl_rens(dsn)/des="use with R SASXport for long variable names";                 
                                                                                            
      if _n_=0 then do;                                                                     
                                                                                            
        rc=%sysfunc(dosubl('                                                                
            data __ren001;                                                                  
               set &dsn(obs=1);                                                             
            run;quit;                                                                       
            proc transpose data=__ren001 out=__ren002(drop=col1);                           
              var _all_;                                                                    
            run;quit;                                                                       
            proc sql;                                                                       
              select                                                                        
                catx(" ",_name_,"as",translate(lbl,"_",".")) into :rens separated by ","    
              from                                                                          
                (                                                                           
                 select                                                                     
                    _name_                                                                  
                   ,case                                                                    
                        when (_label_ = " ") then _name_                                    
                        else _label_                                                        
                    end as lbl                                                              
                 from                                                                       
                    __ren002                                                                
                )                                                                           
           ;quit;                                                                           
            proc sql;                                                                       
               create                                                                       
                   view %scan(&dsn,2,".")  as                                               
               select                                                                       
                   &rens                                                                    
               from                                                                         
                   &dsn.                                                                    
            ;quit;                                                                          
        '));                                                                                
        drop rc;                                                                            
      end;                                                                                  
                                                                                            
    %mend utl_rens;                                                                         
                                                                                            



    libname xpt xport "d:/xpt/rxpt.xpt";
    data iris_log_names;

      %utl_rens(xpt.iris);
      set iris;

    run;quit;



