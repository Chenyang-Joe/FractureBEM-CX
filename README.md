# FractureBEM

Sources for the paper High-Resolution Brittle Fracture Simulation with Boundary Elements

This project has been replaced by https://github.com/david-hahn/FractureRB.

# My deployment

1. **Build the Docker Image**
```bash
docker build -t fracture-bem . 
```

2. **Enter the Docker Container**
```bash
docker run -it -v "$PWD":/app \
-w /app \
fracture-bem bash
```


3. **Build and Compile the Project**
```bash
mkdir -p build && cd build
cmake .. \
-DCMAKE_CXX_FLAGS="-D_GLIBCXX_USE_CXX11_ABI=1" \
-DCMAKE_BUILD_TYPE=Debug \
-DHLIB_INC=/usr/include/eigen3 \
-DBUILD_HYENA=OFF   \
-DHYENA_LIB=/app/hyena/libHyENAlib2.so \
-Dzlib=/usr/lib/x86_64-linux-gnu/libz.so \
-DHalflib=/usr/lib/x86_64-linux-gnu/libHalf.so \
-DOpenVDBlib=/usr/local/lib/libopenvdb.so \
-DOpenVDBinclude=/usr/local/include

make -j$(nproc)
```

4. **Run the Example**
```bash
cd ../examples
chmod +x column_run.sh
./column_run.sh
run
bt
```
Or try ```bunny_mat_run.sh``` which shows the same error.


# Debug Info
According to the debug info #5, the elem = 4294966437 in FractureSim::FractureBEM::checkDistanceEl. It seems that signed-to-unsigned conversion happens to "elem" somewhere, so elem has such a big value. e.g. -155 -> 4294966437. When I run bunny_mat_run.sh, it also shows the same type of error.


(gdb) bt
#0  0x00005555559f3564 in std::less<unsigned int>::operator() (this=0x7fffffffd028, __x=@0x5555564aa8a0: 127, __y=<error reading variable>)
    at /usr/include/c++/8/bits/stl_function.h:386
#1  0x0000555555a2222f in std::_Rb_tree<unsigned int, std::pair<unsigned int const, std::vector<double, std::allocator<double> > >, std::_Select1st<std::pair<unsigned int const, std::vector<double, std::allocator<double> > > >, std::less<unsigned int>, std::allocator<std::pair<unsigned int const, std::vector<double, std::allocator<double> > > > >::_M_lower_bound (this=0x7fffffffd028, __x=0x5555564aa880, __y=0x7fffffffd030, __k=<error reading variable>)
    at /usr/include/c++/8/bits/stl_tree.h:1888
#2  0x0000555555a1ceb3 in std::_Rb_tree<unsigned int, std::pair<unsigned int const, std::vector<double, std::allocator<double> > >, std::_Select1st<std::pair<unsigned int const, std::vector<double, std::allocator<double> > > >, std::less<unsigned int>, std::allocator<std::pair<unsigned int const, std::vector<double, std::allocator<double> > > > >::lower_bound (this=0x7fffffffd028, __k=<error reading variable>) at /usr/include/c++/8/bits/stl_tree.h:1203
#3  0x0000555555a169fd in std::map<unsigned int, std::vector<double, std::allocator<double> >, std::less<unsigned int>, std::allocator<std::pair<unsigned int const, std::vector<double, std::allocator<double> > > > >::lower_bound (this=0x7fffffffd028, __x=<error reading variable>)
    at /usr/include/c++/8/bits/stl_map.h:1239
#4  0x0000555555a0df98 in std::map<unsigned int, std::vector<double, std::allocator<double> >, std::less<unsigned int>, std::allocator<std::pair<unsigned int const, std::vector<double, std::allocator<double> > > > >::operator[] (this=0x7fffffffd028, __k=<error reading variable>)
    at /usr/include/c++/8/bits/stl_map.h:495
--Type <RET> for more, q to quit, c to continue without paging--
#5  0x0000555555a031f7 in FractureSim::FractureBEM::checkDistanceEl (this=0x7fffffffd020, elem=4294966437, nodeSet=std::set with 30 elements = {...})
    at /app/src/FractureBEM.cpp:562
#6  0x0000555555a000d6 in FractureSim::FractureBEM::seedCracksAndPropagate (this=0x7fffffffd020, maxSeed=17) at /app/src/FractureBEM.cpp:234
#7  0x00005555559e49e8 in main (argc=32, argv=0x7fffffffeb88) at /app/src/main.cpp:218