Template Model Builder (TMB)
============================

This fork adds control to the compile() function (found inside TMB.R) to obtain full control over the mingw64 g++ '-Wall' (warnings all) argument under Windows.

See TMB issue 321  ( https://github.com/kaskr/adcomp/issues/321#issuecomment-1022628013 )

There is a new argument, 'remove_arg_Wall' (default TRUE) in the compile() function and the changes are around lines 1,100 to 1,116:


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
                     CXXFLAGS=sub(" $","", paste(Makeconf_args_global,flags[flags!=""])), ## Now flags overrides only the Makeconf cxxflags that are the same. [Need paste here, not 'c()'.]
                     ...
                     )     
        
Note that in my experience, the '-Wall' argument doesn't always produce excess warnings, however removing the '-Wall' argument never shows the excess warnings.  
       
Now the currrent R version's global 'Makeconf' [at location: paste0(R.home("etc"), "/x64/Makeconf")] 'CXXFLAGS' contents are no longer replaced by the flags contents 
in the temporaily created 'R_MAKEVARS_USER'. The 'Makeconf' 'CXXFLAGS' contents are scan()'ed in and only the compile()'s 'flags' argument(s) that are the same override the 'Makeconf' 'CXXFLAGS' contents. It does this by having the 'flags' contents being more to the right in the 'g++' call than the 'Markeconf' 'CXXFLAGS' flags. (See the code.) This can leave a few vestigil flags in the call that are not used. If important flags, now or in the future, are in the 'Makeconf' 'CXXFLAGS' contents they are no longer lost when extra arguments are added using the 'flags' argument.
    
   
    
Please test this fork and report any issues, thanks.     

Extra info




