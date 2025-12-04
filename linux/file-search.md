# File & Text Search Commands

## Find Missing Files

### Locate (Database Search)
```bash
locate <filename>
```

### Find (Traversal Search)
```bash
find / -name "<filename>"
find /path -type f -iname "*.log"
```

## Search Inside Files
### Grep (Recursive/Text Search)
```bash
grep -r "<pattern>" /path
grep -rnw . -e "<pattern>"
```

## Debug Text Manipulation
### Cut / Awk / Sed
```bash
awk -F':' '{print $1}' file.txt
cut -d " " -f 1,2 file.txt
sed -i 's/old/new/g' file.txt
```

## Cleanup & Formatting
```bash
tr -d '\r' <file> > clean.txt
sort -u input.txt
```