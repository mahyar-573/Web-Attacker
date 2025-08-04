#!/bin/bash

# تنظیمات رنگ برای خروجی
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

# دریافت اطلاعات هدف
echo -e "${YELLOW}[*] Enter Target IP/Host:${NC} "
read target
echo -e "${YELLOW}[*] Enter Web Port (default 80):${NC} "
read web_port
web_port=${web_port:-80} # مقدار پیش‌فرض 80

# ایجاد پوشه برای نتایج
report_dir="Pentest_Report_$(date +%Y%m%d_%H%M%S)"
mkdir -p "$report_dir"
echo -e "${GREEN}[+] Results will be saved in: $report_dir${NC}"

# تابع برای اجرای دستورات و ذخیره خروجی
run_test() {
    echo -e "${YELLOW}[*] Running: $1${NC}"
    eval "$2" > "$report_dir/$3"
    echo -e "${GREEN}[+] Saved: $report_dir/$3${NC}"
}

# 1. اسکن Nmap (پورت‌ها و سرویس‌ها)
run_test "Nmap Scan" "nmap -sV -A -T4 -p- $target" "nmap_scan.txt"

# 2. تست وب با Curl و Nikto
if grep -q "$web_port/open" "$report_dir/nmap_scan.txt"; then
    run_test "Curl HTTP Headers" "curl -I http://$target:$web_port" "curl_headers.txt"
    run_test "Nikto Scan" "nikto -h http://$target:$web_port" "nikto_scan.txt"
    
    # 3. Bruteforce با Hydra (اگر SSH/POSTGRES/MYSQL باز بود)
    if grep -q "22/open" "$report_dir/nmap_scan.txt"; then
        run_test "Hydra SSH Attack" "hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://$target -t 4 -V" "hydra_ssh.txt"
    fi
fi

# 4. تست Metasploit (بررسی آسیب‌پذیری‌های شناخته شده)
run_test "Metasploit Vulnerability Scan" "msfconsole -q -x 'use auxiliary/scanner/http/http_version; set RHOSTS $target; set RPORT $web_port; run; exit'" "metasploit_scan.txt"

# 5. تست SQL Injection (اگر پورت وب باز بود)
if grep -q "$web_port/open" "$report_dir/nmap_scan.txt"; then
    run_test "SQLMap Scan" "sqlmap -u 'http://$target:$web_port/login' --batch --crawl=1" "sqlmap_scan.txt"
fi

# جمع‌بندی
echo -e "${GREEN}\n[+] Penetration Test Completed!${NC}"
echo -e "${YELLOW}=============================================${NC}"
echo -e "${GREEN}نتایج در پوشه '$report_dir' ذخیره شد:${NC}"
ls -l "$report_dir"
