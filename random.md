uname -m && sw_vers -productVersion && sysctl -n machdep.cpu.brand_string 2>/dev/null; system_profiler SPHardwareDataType | grep -E "Chip|Processor|Memory"


git clone https://github.com/PrismML-Eng/Bonsai-demo.git
cd Bonsai-demo
./setup.sh
./scripts/start_llama_server.sh




ollama pull aratan/Ternary-Bonsai-2-27B-gguf:TQ1_0

ollama run aratan/Ternary-Bonsai-2-27B-gguf:TQ1_0
