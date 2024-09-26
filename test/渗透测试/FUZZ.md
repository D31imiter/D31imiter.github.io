##### FUFF

ffuf -c -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -u '192.168.108.143/~secret/.FUZZ' -e .txt,.html -fc 403
