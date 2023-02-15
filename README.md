# 

# s800GRETINA
This unpacker is a modification of GRETINA-LBNL/gretina-unpack for GRETINA + S800 written H. Crawford.
## Installation
```
git clone https://github.com/rajghim/s800GRETINA.git
cd s800GRETINA
Configure the config.json file
mkdir -p build && cd build
cmake ..
make
```

## Running
After installing, run the executable with necessary flags
```
./s800GRETINA -g -zip -d
```
