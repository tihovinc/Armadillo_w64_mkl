#### Build steps:

1. Build SuperLU (read how_to_build.md from https://github.com/tihovinc/SuperLU_w64_mkl.git)
2. Build Armadillo (read how_to_build.md from https://github.com/tihovinc/armadillo-ng_w64_mkl.git
3. Download and install HDF5
4. Download and unpack Armadillo source (version 7.400.2 is included in this repository, for 
   newer check http://arma.sourceforge.net/download.html)
5. Open CMD, call VS2015x64NativeCmdWithMKL.bat, cd into armadillo folder and then type:

  cmake -D CMAKE_LIBRARY_PATH="%ARMADILLOROOT%\lib;%SUPERLUROOT%\lib" -D CMAKE_INCLUDE_PATH="%SUPERLUROOT%\Include" -D BUILD_SHARED_LIBS=false -D DETECT_HDF5=true -G "Visual Studio 14 2015 Win64" .  
  (if you want to compile dll instead of lib than replace string lib with dlllib and BUILD_SHARED_LIBS=false with BUILD_SHARED_LIBS=true in previous line)  

6. Open armadillo.sln from build folder
7. Select all projects
8. Right click -> Properties
9. Select All Configurations in drop down control Configuration
10. Choose Configuration Properties -> General
11. Edit Output Directory field to contain $(SolutionDir)Out\$(Configuration)\
12. Edit Intermediate Directory field to contain $(SolutionDir)Int\$(Configuration)\$(ProjectName)\
13. If you want to be able to execute executables on Windows XP than change Platform Toolset to v140_xp
14. If you want to bulid dll project add ARMADILLO_DLL;H5_BUILT_AS_DYNAMIC_LIB to C++ preprocessor definitions for armadillo project
15. Open config.hpp from tmp\include subfolder of armadillo folder and change #define ARMA_AUX_LIBS into this:

  #ifdef ARMADILLO_DLL  
  #define ARMA_AUX_LIBS C:/Program Files (x86)/IntelSWTools/compilers_and_libraries/windows/mkl/lib/intel64/mkl_rt.lib;C:/Program Files/HDF_Group/HDF5/1.10.0/lib/hdf5.lib;C:/Program Files/HDF_Group/HDF5/1.10.0/lib/hdf5_cpp.lib;C:/Program Files/Armadillo/dlllib/armadillo.lib;C:/Program Files/SuperLU/dlllib/superlu.lib  
  #else  
  #define ARMA_AUX_LIBS C:/Program Files (x86)/IntelSWTools/compilers_and_libraries/windows/mkl/lib/intel64/mkl_rt.lib;C:/Program Files/HDF_Group/HDF5/1.10.0/lib/libhdf5.lib;C:/Program Files/HDF_Group/HDF5/1.10.0/lib/libhdf5_cpp.lib;C:/Program Files/Armadillo/lib/armadillo.lib;C:/Program Files/SuperLU/lib/superlu.lib  
  #endif  

16. If you are building dll project than remove from linker input everything related to mkl and add instead C:\Program Files (x86)\IntelSWTools\compilers_and_libraries\windows\mkl\lib\intel64\mkl_rt.lib
17. If you are building dll project than add armadillodll.def in Configuration Properties -> Linker -> Input -> Module Definition File (its creation is described in next section)
18. Build armadillo project  
19. Edit config.hpp from tmp\include subfolder of armadillo folder and undefine ARMA_USE_WRAPPER
19. Run Administrative cmd, cd into Out subfolder in solution folder and execute:
    
  setx ARMADILLOROOT "%ProgramFiles%\Armadillo" /M  
  set ARMADILLOROOT=%ProgramFiles%\Armadillo   
  xcopy /E /Q /R /Y tmp\include\* "%ARMADILLOROOT%\include  
  :: And if you built static lib than next 2 lines:  
  md "%ARMADILLOROOT%\lib"  
  copy /B /Y armadillo.lib "%ARMADILLOROOT%\lib"  
  :: and if you built dll than next 5 lines:  
  md "%ARMADILLOROOT%\lib"  
  md "%ARMADILLOROOT%\bin"  
  copy /B /Y Out\$(Configuration)\armadillo.lib "%ARMADILLOROOT%\lib\armadillodll.lib"  
  copy /B /Y Out\$(Configuration)\armadillo.dll "%ARMADILLOROOT%\bin\armadillo.dll"  
  and add %ARMADILLOROOT%\bin to path if it isn't already added (setx PATH "%PATH%%ARMADILLOROOT%\bin;")  


#### How to create def file for dll:
1. Build static lib for at least one configuration
2. cd into out dir for this configuration and execute:

  dumpbin /linkermember:2 armadillo.lib > armadillodll.def  
  
3. Edit out from def file everything except function names and add def header at the beginning of file:

  LIBRARY armadillo  
  EXPORTS      
  (Notepad++ with its ability for block selection will make it a 1 minute job) 

4. Save file
  