# map

1. map容器不允许有多个key值（允许有多个value），若多次插入相同key值的pair，则取第一个插入的pair的值

   ~~~ c++
   int main()
   {
   	map<int, int>m;
   	m.insert(make_pair(1, 10));
   	m.insert(pair<int,int>(2, 20));
   	m.insert(make_pair(2, 30));
   	for (map<int, int>::const_iterator it = m.cbegin(); it != m.cend(); it++)
   	{
   		cout << "key: " << it->first << endl << "value: " << it->second << endl << endl;
   	}
   	return 0;
   }
   ~~~

2. map容器插入会自动按照key值排序

   ~~~ c++
   int main()
   {
   	map<int, int>m;
   	m.insert(make_pair(1, 10));
   	m.insert(pair<int,int>(2, 20));
   	m.insert(make_pair(6, 30));
   	m.insert(make_pair(3, 30));
   	for (map<int, int>::const_iterator it = m.cbegin(); it != m.cend(); it++)
   	{
   		cout << "key: " << it->first << endl << "value: " << it->second << endl << endl;
   	}
   	return 0;
   }
   ~~~

3. ~~~ c++
   int main()
   {
   	map<int, int>m;
   	m.insert(make_pair(1, 10));
   	m.insert(make_pair(2, 20));
   	m.insert(make_pair(6, 60));
   	m.insert(make_pair(3, 30));
   	map<int, int>::iterator it = m.begin();
   	it++;
   	it++;
       m.erase(2);//删除该键值对应的pair
   	m.erase(it);//删除迭代器所在位置元素，返回下一个位置的迭代器
   	Print_Map(m);
   	return 0;
   }
   ~~~

4. ~~~ c++
   int main()
   {
   	map<int, int>m;
   	m.insert(make_pair(1, 10));
   	m.insert(make_pair(2, 20));
   	m.insert(make_pair(6, 60));
   	m.insert(make_pair(3, 30));
   	map<int, int>::iterator it = m.begin();
   	cout << m.find(6)->first << endl;//返回该key的迭代器
   	cout << m.find(6)->second << endl;
   	cout << m.count(2) << endl;//返回map中key值pair的个数
   	return 0;
   }
   ~~~