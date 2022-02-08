Template Model Builder (TMB)
============================
This fork addresses the TMB Issue 321  ( https://github.com/kaskr/adcomp/issues/321#issuecomment-1022628013 ) of excess warnings when TMB is running under Windows.

I have added flag: "-Wno-ignored-attributes" to the compile() function to eliminate the warnings when needed. See the comments in the code snippet below which is setting up options for the 'g++' call:


      useRcppEigen <- !file.exists( system.file("include/Eigen",package="TMB") )  # TRUE when TMB/include/Eigen does not exist.
      useContrib   <-  file.exists( system.file("include/contrib",package="TMB") )
      ppflags <- paste(paste0("-I",qsystem.file("include",package="TMB")),
               # The CRAN install of TMB doesn't install the "Eigen" library (nor the TMBad library) in TMB/include.
               # When there is no 'Eigen' library, the next line instructs the 'g++' call to use the 'RcppEigen' package instead.
               paste0("-I",qsystem.file("include",package="RcppEigen"))[useRcppEigen], 
               -Wno-ignored-attributes"[useRcppEigen], # Removes excess warnings under Windows when the 'RcppEigen' package is used.
               paste0("-I",qsystem.file("include/contrib",package="TMB"))[useContrib],
               ...
              
When the "Eigen" library doesn't exist, there are now two parts added to the 'g++' call; previously only the first option was added.
    
     -I"W:/R/R-4.1.2/library/RcppEigen/include" -Wno-ignored-attributes
              
                       
 Installing TMB from GitHub does add both the "Eigen" and "TMBad" libraries to TMB/include.
 
This change is compatible with TMB running under R on Linux.


As a side note, using the '-w' flag (note the lower case) in the flags argument of compile() will remove all warnings, e.g.:

    compile('linreg_parallel.cpp', '-w')

Of course, such a global removal of warnings needs to used with caution.
