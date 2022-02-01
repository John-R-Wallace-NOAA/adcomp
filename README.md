Template Model Builder (TMB)
============================

This fork adds control to the compile() function (found inside TMB.R) to obtain full control over the mingw64 g++ '-Wall' (warnings all) argument under Windows.

See TMB issue 321  ( https://github.com/kaskr/adcomp/issues/321#issuecomment-1022628013 )

There is a new argument, 'remove_arg_Wall' (default TRUE) in the compile() function and the changes are on lines 1,100 to 1,108:


    Makeconf_file <- paste0(R.home("etc"), "/x64/Makeconf")
    if(file.exists(Makeconf_file)) {
        Makeconf <- scan(Makeconf_file, what = "", sep = "\n", quiet = TRUE)
        if(remove_arg_Wall)
           Makeconf_args_no_Wall <- sub("-Wall", "", sub("CXXFLAGS = ", "", Makeconf[grep("^CXXFLAGS", Makeconf)]))
        else
          Makeconf_args_no_Wall <- sub("CXXFLAGS = ", "", Makeconf[grep("^CXXFLAGS", Makeconf)])
     } else
        Makeconf_args_no_Wall <- character(0)
        
Note that in my experience, the '-Wall' argument doesn't always produce excess warnings, however removing the '-Wall' argument never shows the excess warnings.  
       
Now the compile()'s argumnet 'flags' overrides only the Makeconf cxxflags that are the same. It does this by being more to the right than the Markeconf flags.
        
Please test this fork and report any issues, thanks.        
