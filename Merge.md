```bash
git init
echo "1">1.txt
git add .
git commit -m "1"
git checkout -b b1
echo "2">2.txt
git add .
git commit -m "2"
git checkout master
echo "3">3.txt
git add .
git commit -m "3"
git merge b1
```
