# Template

函数模板在调用时才会生成，不调用该函数模板将不会出现报错（模板调用了不同类的成员函数）

~~~ c++
class Person1 {
public:
	void ShowPerson1(){
		cout << "Person1";
	}
};
class Person2 {
public:
	void ShowPerson2() {
		cout << "Person2";
	}
};
template<typename T>
void test(T a)
{
	a.ShowPerson1();
	a.ShowPerson2();
}
int main()
{
	Person1 p;
	//test<Person1>(p);
	return 0;
}
~~~

