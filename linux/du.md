du：disk use
df：disk free
Summarize disk usage of each FILE, recursively for directories.

1. df -lh

2. du -s ./* | sort -rn 
这是按字节排序

3. du -sh ./* | sort -rn 
这是按兆（M）来排序

4.选出排在前面的10个 
du -s ./* | sort -rn | head

5.选出排在后面的10个 
du -s ./* | sort -rn | tail