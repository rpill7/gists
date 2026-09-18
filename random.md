uname -m && sw_vers -productVersion && sysctl -n machdep.cpu.brand_string 2>/dev/null; system_profiler SPHardwareDataType | grep -E "Chip|Processor|Memory"
