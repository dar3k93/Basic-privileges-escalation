#### Plain text usernames and/or password
- grep -i user [filename]
- grep -i pass [filename]
- grep -C 5 "password" [filename]
- find . -name "*.php" -print0 | xargs -0 grep -i -n "var $password"   # Joomla
