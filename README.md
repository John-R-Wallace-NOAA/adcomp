Template Model Builder (TMB)
============================

This fork adds control to the compile() function (found inside TMB.R) to obtain full control over the mingw64 g++ '-Wall' (warnings all) argument under Windows.

See TMB issue 321  ( https://github.com/kaskr/adcomp/issues/321#issuecomment-1022628013 )

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
        
In my experience, the '-Wall' argument doesn't always produce excess warnings, however removing the '-Wall' argument never shows the excess warnings.  
       
Note that the current R version's global 'Makeconf' [at location: paste0(R.home("etc"), "/x64/Makeconf")] 'CXXFLAGS' contents are no longer replaced by the compile()'s 'flags' contents.  The 'Makeconf' 'CXXFLAGS' contents are scan()'ed in and only the compile()'s 'flags' argument(s) that are the same override the 'Makeconf' 'CXXFLAGS' contents. This is done by having the 'flags' contents being more to the right than the 'Markeconf' 'CXXFLAGS' flags in the 'g++' call. (This can leave a few vestigial flags in the call that go unused.)   If important flags, now or in the future, are in 'Makeconf's 'CXXFLAGS' they are no longer lost when extra arguments are added using the 'flags' argument.
    
Setting remove_arg_Wall = FALSE will not remove the '-Wall' flag.   
       
Please test this fork and report any issues, thanks.     

---

## Extra info

I haven't actually done this (except that flags = " " does work); however it has now occurred to me that:

To have the compile()'s 'flags' argument continue to replace all the 'Makeconf' 'CXXFLAGS'contents, and have the default be to use none of the 'Makeconf' 'CXXFLAGS' contents, thus accomplishing the goal of getting rid of the '-Wall' flag, then that could be accomplished in 2 steps. 

1) Add an argument 'Makeconf_global' to compile() with a default of FALSE. 

2) Add the following two lines of code to the compile():


           if(flags=="" & !Makeconf_global)
               flags <- " "

## High strangeness

Submitting these 'g++' calls to a Windows 10 Command Window (cmd) and thus not using R, I have found the follwing:

This call (from an older version of TMB) almost always gives excess warnings as one would expect with the '-Wall' flag:

    "C:/rtools40/mingw64/bin/"g++ -std=gnu++11  -I"W:/MKL/MKL/include" -DNDEBUG -I"W:/MKL/MKL/library/TMB/include"   -DTMB_SAFEBOUNDS -DLIB_UNLOAD=R_unload_simple  -DTMB_LIB_INIT=R_init_simple         -O2 -Wall  -mfpmath=sse -msse2 -mstackrealign -c simple.cpp -o simple.o
    
This call, which still has the '-Wall' flag, with only an extra space between '-mstackrealign' and '-c', almost never gives excess warnings:
    
    "C:/rtools40/mingw64/bin/"g++ -std=gnu++11  -I"W:/MRO/MRO/include" -DNDEBUG -I"W:/MRO/MRO/library/TMB/include"   -DTMB_SAFEBOUNDS -DLIB_UNLOAD=R_unload_simple  -DTMB_LIB_INIT=R_init_simple          -O2 -Wall  -mfpmath=sse -msse2 -mstackrealign  -c simple.cpp -o simple.o
    
This call with reduced spaces more randomly sometimes gives excess warnings and sometimes not, when intermixed with other calls:

    "C:/rtools40/mingw64/bin/"g++ -std=gnu++11  -I"W:/MRO/MRO/include" -DNDEBUG -I"W:/MRO/MRO/library/TMB/include"   -DTMB_SAFEBOUNDS -DLIB_UNLOAD=R_unload_simple  -DTMB_LIB_INIT=R_init_simple          -O2 -Wall  -mfpmath=sse -msse2 -mstackrealign  -c simple.cpp -o simple.o



