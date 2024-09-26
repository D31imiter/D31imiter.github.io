# c++

# static

1. 静态成员，被static修饰的成员变量或函数，可以通过对象，对象的指针，类来访问

2. 静态成员变量，储存在全局区，运行过程中只有一份内存

3. 对比全局变量，可以通过权限来达到局部共享的目的

4. 必须初始化,如下：如果类的声明和实现分离，初始化时不带static

   ~~~ c++
   class student{
       static int a；
   };
   int student:: a = 6;
   ~~~

   

5. 静态成员函数，内部不能使用this指针，this指针只能用在非静态成员函数内部

   1. 不能是虚函数，虚函数只能是非静态成员函数

   2. 内部不能访问非静态成员变量函数，只能访问静态成员函数

   3. 非静态成员函数可以访问静态成员函数变量

   4. 构造函数析构函数不能是静态函数

      

# 单例模式

设计模式的一种，保证某个类永远只能创建一个对象

1. 构造函数私有化

2. 定义一个私有的static成员变量指向唯一的单例对象

3. 提供一个公共的访问单例对象的接口

   ~~~ c++
   class Rocket {
   private:
   	Rocket(){}
   	static Rocket* ms_rocket;
   public:
   	static Rocket* sharerocket() {
   		if (ms_rocket == NULL) {
   			ms_rocket = new Rocket();
   		}
   		return ms_rocket;
   	}
   	void run() {
   		cout << "run" << endl;
   	}
   };
   Rocket* Rocket::ms_rocket = NULL;//静态成员变量必须初始化
   int main() {
   	Rocket* p1 = Rocket::sharerocket();
   	return 0;
   }
   ~~~

   

# const

1. const成员：被const修饰的成员变量，非静态成员函数

2. const成员变量必须初始化，可以创建时初始化，非静态的const成员变量也可以通过初始化列表来初始化

3. const成员函数

   1. const关键字写在参数列表后边，函数的声明和实现都必须带const

   2. 内部不能修改非static成员变量

   3. 内部只能调用const，static成员函数，非const成员函数可以调用const成员函数

   4. const成员函数与非const成员数函数构成重载

   5. 非const成员（指针）优先调用非const成员函数

   6. const成员函数只能调用const、static成员函数

      

~~~ c++
class person{
public:
	const int m_age=6;
	person():m_age(10) {
	}
};
int main()
{
	person person;
	cout << person.m_age << endl;
	return 0;
}
~~~

# 拷贝构造函数

~~~ c++
class person {
public:
	int m_age ;
	int m_height;
	person():m_age(10), m_height(100) {
	}
	person(const person& person) {
		cout << "const person& person" << endl;
	}
};
int main()
{
	person person1;

	person person2(person1);
	
	cout << person2.m_age << endl << person2.m_height << endl;

	return 0;
}
~~~

1. 编译器默认的提供的拷贝是浅拷贝

   1. 将一个对象中所有的成员变量的值拷贝到另一个对象

   2. 如果某个成员变量是个指针，只会拷贝指针中储存的地址值，并不会拷贝指针指向新的内存空间

   3. 可能会导致多次free的问题

      

2. 若果要实现深拷贝，就要自定义拷贝构造函数，将指针变量所指向的内存空间，拷贝到新的内存空间

   # 运算符重载
   
   ~~~ c++
   class Vector {
   private:
   	enum Mode;
   	double m_x;
   	double m_y;
   	double m_length;
   	double m_angle;
   	Mode mode;
   	void Set_x(double x) {
   		this->m_x = x;
   	}
   	void Set_y(double y) {
   		this->m_y = y;
   	}
   	void Set_length(double length) {
   		this->m_length = length;
   	}
   	void Set_angle(double angle) {
   		this->m_angle = angle;
   	}
   public:
   	enum Mode { rect, pol };
   	Mode Get_Mode() {
   		return this->mode;
   	}
   	Vector():m_x(0), m_y(0), m_length(0), m_angle(0), mode(rect){}
   	void Set_Mode(Mode o_mode) {
   		this->mode = o_mode;
   	}
   	Vector(const Vector& v) {
   		*this = v;
   	}
   	void Inital(double x,double y,Mode form) {
   		if (form == rect){
   			this->mode = form;
   			this->Set_x(x);
   			this->Set_y(y);
   			this->Set_length(sqrt(this->m_x * this->m_x + this->m_y * this->m_y));
   			this->Set_angle(acos(this->m_x/this->m_length));
   		}else if(form == pol){
   			this->mode = form;
   			this->Set_length(x);
   			this->Set_angle(y);
   			this->Set_x(this->m_length * cos(this->m_angle));
   			this->Set_y(this->m_length * sin(this->m_angle));
   		}
   	}
   	Vector operator+(const Vector & v) {
   		Vector ret;
   		ret.Inital(v.m_x + this->m_x, v.m_y + this->m_y, this->rect);
   		return ret;
   	}
   	Vector operator-(const Vector& v) {
   		Vector ret;
   		ret.Inital(v.m_x - this->m_x, v.m_y - this->m_y, this->mode);
   		return ret;
   	}
   	double operator*(const Vector& v) {
   		return v.m_length * this->m_length * (cos(abs(v.m_angle - this->m_angle)));
   	}
   	Vector operator*(double num) {
   		Vector ret;
   		ret.Inital(this->m_x * num, this->m_y * num,this->mode);
   		return ret;
   	}
   	friend Vector operator*( double num, Vector v);
   	friend ostream& operator<<(ostream& os, Vector & v);
   };
   Vector operator*(double num,Vector v) {
   	Vector ret;
   	return v * num;
   }
   ostream& operator<<(ostream& os, Vector & v) {
   	if (v.Get_Mode() == Vector::rect) {
   		cout << "x: " << v.m_x << endl << "y: " << v.m_y << endl;
   	}
   	else if (v.Get_Mode() == Vector::pol) {
   		cout << "length: " << v.m_length << endl << "angle: " << v.m_angle << endl;
   	}
   	return os;
   }
   int main()
   {
   	Vector v1;
   	v1.Inital(10, 20, Vector::pol);
   	Vector v2;
   	v2.Inital(10, 20, Vector::pol);
   	Vector v3 = v1 + v2;
   	v3.Set_Mode(Vector::rect);
   	cout << v1 << v2 << v3;
   	return 0;
   }
   ~~~
   
   # 隐式构造
   
   ~~~ c++
   class person {
   public:
   	int m_age;
   	person() {
   		cout << "person()" << endl;
   	}
   	person(int age) :m_age(age) {
   		cout << "person(int age):m_age(age)" << endl;
   	}
   	~person(){
   		cout << "~person()" << endl;
   }
   };
   int main()
   {
   	person p1;
   	p1 = 50; 
   	//person()：p1创建时调用的构造函数
   	//person(int age) :m_age(age)：赋值时产生了不必要的临时对象，调用了有参的构造函数，
   	//~person()：临时对象被创建完之后就立刻销毁
   	//~person()：main函数结束后p1被销毁
   	person p2 = 50;
   	//相当于调用了有参的构造函数，等同person p2(50);
   	return 0;
   }
   ~~~
   
   # explicit
   
   声明该函数禁止调用隐式构造
   
   ~~~ c++
   class person {
   public:
   	int m_age;
   	person() {
   		cout << "person()" << endl;
   	}
   	explicit person(int age) :m_age(age) {//explicit声明该函数禁止调用隐式构造
   		cout << "person(int age):m_age(age)" << endl;
   	}
   	~person(){
   		cout << "~person()" << endl;
   	}
   };
   int main()
   {
   	person p1;
   	p1 = 50;//隐式构造被禁止调用，报错
   	return 0;
   }
   ~~~
   
   # 友元函数
   
   friend+函数声明，使该函数成为该对象的友元，友元函数可以访问私有成员
   
   ~~~ c++
   class Point {
   	friend Point add(Point, Point);
   	int m_x;
   	int m_y;
   public:
   	int get_x() { return m_x; }
   	int get_y() { return m_y; }
   	Point(int x,int y):m_x(x),m_y (y){}
   	void display() {
   		cout << "(" << m_x << "," << m_y << ")" << endl;
   	}
   };
   Point add(Point p1, Point p2) {
   	return Point(p1.m_x + p2.m_x, p1.m_y + p2.m_y);
   }
   int main()
   {
   	Point p1(10, 20);
   	Point p2(20, 30);
   	Point p3 = add(p1, p2);
   	p3.display();
   	return 0;
   }
   ~~~
   
   

