Template Model Builder (TMB)
============================
This fork addresses the TMB Issue 321  ( https://github.com/kaskr/adcomp/issues/321#issuecomment-1022628013 ) of excess warnings when TMB is running under Windows.

## The flag: < -Wno-ignored-attributes > added to the 'flags' argument under Windows

The 'g++' compile() call includes the RcppEigen package libraries: -I".../RcppEigen/include" which causes warnings about ignoring attributes on template arguments. The '-Wno-ignored-attributes' flag stops those warnings and is added to the beginning of the 'flags' argument early in the compile() function when run under Windows. (If one is nostalgic for a screen full of warnings, they can be temporarily reinstated with < flags = "-Wignored-attributes" > in the compile() call.)

The issue with having the 'flags' argument's default not being an empty character string later in the compile() function, is that the R global Makeconf's CXXFLAGS would always be overwritten; this issue is fixed below.  Note that this is increased functionality, since currently any flag given to the 'flags' argument removes all the global Makeconf's CXXFLAGS flags. (The Makeconf's CXXFLAGS flags could have been manually re-added to the 'flags' argument.)

## Full control over R's global Makeconf's CXXFLAGS entry under Windows 

There is a new argument, 'del_args_Makeconf' (default "-Wall") in the compile() function and the following changes are made around lines 1,100 to 1,116 of the TMB.R file:


       Makeconf_file <- paste0(R.home("etc"), Sys.getenv("R_ARCH"), "/Makeconf")
       if(file.exists(Makeconf_file)) {
          Makeconf <- scan(Makeconf_file, what = "", sep = "\n", quiet = TRUE)
          Makeconf_args_global <- sub("CXXFLAGS = ", "", Makeconf[grep("^CXXFLAGS", Makeconf)])
          if(del_args_Makeconf!="") {
             for(i in del_args_Makeconf)
              Makeconf_args_global <- sub(i, "", Makeconf_args_global)
          }
       } else
          Makeconf_args_global <- character(0)
       mvfile <- makevars(PKG_CPPFLAGS=ppflags, PKG_LIBS=paste("$(SHLIB_OPENMP_CXXFLAGS)"[openmp] ),
                          PKG_CXXFLAGS="$(SHLIB_OPENMP_CXXFLAGS)"[openmp],
                          CXXFLAGS=paste(Makeconf_args_global,flags[flags!=""]), ## Now flags overrides only the Makeconf cxxflags that are the same. [Need paste here, not 'c()'.]
                          ...
                          ) 
  
  
- The 'Makeconf' 'CXXFLAGS' contents are scan()'ed in and flags equal to those in the 'del_args_Makeconf' argument are removed. 
- The compile()'s 'flags' argument(s) that are the same override the 'Makeconf' 'CXXFLAGS' contents, e.g. '-O1' in 'flags' would override the default '-O2' flag. This is done by having the 'flags' contents being more to the right than the remaining 'Markeconf' 'CXXFLAGS' flags in the 'g++' call. 
- This can leave a few vestigial flags in the call that go unused. To avoid this, the 'del_args_Makeconf' argument could be set to c("-Wall", "-O2") in the example above.
- Note that if important flags, now or in the future, are in 'Makeconf's 'CXXFLAGS' they are no longer lost when the 'flags' argument is used.
- For standard CRAN R, with the < -Wno-ignored-attributes > flag added, there are currently no excess warnings if < del_args_Makeconf = "" >, however < del_args_Makeconf = "-Wall" > may be needed in the future and so it has been left as the default.
- Also, < del_args_Makeconf = "-Wall" > is needed for R versions for which Intel's MKL libraries have been added (there are no excess warnings in MRO it appears), see: https://github.com/John-R-Wallace-NOAA/R_4.X_MRO_Windows_and_R_MKL_Linux
- The help is not yet updated.

    
Please test this fork and report any issues, thanks.     

---

## Information on the other CXXFLAGS in Makeconf


 -mfpmath is for generating floating point arithmetics for the selected unit (sse): https://gcc.gnu.org/onlinedocs/gcc-4.5.3/gcc/i386-and-x86_002d64-Options.html
 
 -mstackrealign realigns the runtime stack if necessary: https://stackoverflow.com/questions/2386408/qt-gcc-sse-and-stack-alignment


---

## Other notes

Neither suppressWarnings() nor suppressMessages() works to suppress these excess warnings coming from a Windows cmd window with a 'g++' call:

     library('TMB')
     suppressWarnings(compile('simple.cpp')) # Doesn't work
     suppressMessages(compile('simple.cpp')) # Doesn't work
     
And as everyone must have first tried, diverting with sink(..., type = "output") and sink(..., type = "message"), doesn't work either:  

    
     zz <- file("all.Rout", open = "wt")
     sink(zz)
     sink(zz, type = "message")
     compile('simple.cpp')
     
     ## revert output back to the console -- only then access the file!
     sink(type = "message")
     sink()
     file.show("all.Rout")
