Template Model Builder (TMB)
============================

This fork adds functionality to the compile() function (found inside TMB.R) to obtain full control over the mingw64 g++ '-Wall' (warnings all) argument under Windows.

See TMB Issue 321  ( https://github.com/kaskr/adcomp/issues/321#issuecomment-1022628013 )

There is a new argument, 'remove_arg_Wall' (default TRUE) in the compile() function and the following changes are made around lines 1,100 to 1,116:


    Makeconf_file <- paste0(R.home("etc"), "/x64/Makeconf")
    if(file.exists(Makeconf_file)) {
        Makeconf <- scan(Makeconf_file, what = "", sep = "\n", quiet = TRUE)
        if(remove_arg_Wall)
           Makeconf_args_no_Wall <- sub("-Wall", "", sub("CXXFLAGS = ", "", Makeconf[grep("^CXXFLAGS", Makeconf)]))
        else
          Makeconf_args_no_Wall <- sub("CXXFLAGS = ", "", Makeconf[grep("^CXXFLAGS", Makeconf)])
    } else
        Makeconf_args_no_Wall <- character(0)
    mvfile <- makevars(PKG_CPPFLAGS=ppflags, PKG_LIBS=paste("$(SHLIB_OPENMP_CXXFLAGS)"[openmp] ),
                     PKG_CXXFLAGS="$(SHLIB_OPENMP_CXXFLAGS)"[openmp],
                     CXXFLAGS=sub(" $","", paste(Makeconf_args_global,flags[flags!=""])), ## Now flags overrides only the Makeconf cxxflags that are the same. [Need paste()  here, not 'c()'.]
                     ...
                     )     
  
  
Note that R's global 'Makeconf' [at location: paste0(R.home("etc"), "/x64/Makeconf")] 'CXXFLAGS' contents are no longer replaced by the compile()'s 'flags' contents.  The 'Makeconf' 'CXXFLAGS' contents are scan()'ed in and only the compile()'s 'flags' argument(s) that are the same override the 'Makeconf' 'CXXFLAGS' contents (e.g. '-O1' in 'flags' would override the default '-O2' flag). This is done by having the 'flags' contents being more to the right than the 'Markeconf' 'CXXFLAGS' flags in the 'g++' call. (This can leave a few vestigial flags in the call that go unused.)   If important flags, now or in the future, are in 'Makeconf's 'CXXFLAGS' they are no longer lost when extra arguments are added using the 'flags' argument.
    
Setting < remove_arg_Wall = FALSE > will retain the '-Wall' flag.   
       
Please test this fork and report any issues, thanks.     

---

## Brute force method

To have the compile()'s 'flags' argument continue to replace all the 'Makeconf' 'CXXFLAGS' contents, and to use none of the 'Makeconf' 'CXXFLAGS' contents be the default, thus accomplishing the goal of getting rid of the '-Wall' flag, then that can be accomplished in 2 steps. 

1) Add an argument 'Makeconf_global' to compile() with a default of FALSE in TMB.R. 

2) Add the following two lines of code to compile():


           if(flags=="" & !Makeconf_global)
               flags <- " "

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
