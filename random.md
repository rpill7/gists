uname -m && sw_vers -productVersion && sysctl -n machdep.cpu.brand_string 2>/dev/null; system_profiler SPHardwareDataType | grep -E "Chip|Processor|Memory"


git clone https://github.com/PrismML-Eng/Bonsai-demo.git
cd Bonsai-demo
./setup.sh
./scripts/start_llama_server.sh
