---
title: "[C++] Polynomial 다항식의 연산"
date: 2019-09-29
categories: DEV C++
---

`다항식(Polynomial)의 덧셈과 곱셈에 대해서`

시작하기
===

오늘은, 다항식의 덧셈과 곱셈을 객체지향언어인 C++을 이용해서,
클래스 개념을 통해 조금 더 조직화하고 캡슐화하여 이해를 돕고 추후에 연결리스트를 공부하기 위한 밑거름을 다지도록 하겠다.


전체적 구조
===
클래스는 Term과 이를 이용하여 항의 연속적인 표현을 위해 Polynomial클래스를 정의한다.<br>
우선 클래스의 이름과 그들이 가지는 내용을 보자.

*다음 그림을 보자*


![poly]({{"/asset/poly.png"}})<br>

*이미지출처 : https://www.mathsisfun.com/algebra/polynomials.html*

term은 각각의 항을 의미하고, 이들을 모아서 terms, term끼리 연산구조를 가지고 모여있는 다항식을
polynomial이라고 한다. 
term은 계수(coef)와 지수(exp)로 구성되고 이것이 모이면 termarray 배열의 형태를 가진다.


클래스 구조
===

header파일과 cpp을 분리하여 관리할 수 있지만, 지금은 단순히 각 클래스들이 가지는 의미를 이해하고, <br>
적용하기 위한 밑거름이니 모아서 작성해간다.


```c++
#include <iostream>
using namespace std;

class Polynomial; //전방선언
class Term {
    friend Polynomial; 
    //Polynomial 클래스에서 접근이 가능하다.(전방선언을 하였기 때문에)
private:
    float coef; // 계수
    int exp; // 지수
};

class Polynomial
{
private:
    Term *termArray;
    int capacity; // 배열크기
    int terms; // 0이 아닌 항의 수
public:
    void Print(void);
    void NewTerm(const float theCoeff, const int theExp); 
    Polynomial Add(Polynomial b);
    Polynomial Multiply(Polynomial b);
    Polynomial(void); //생성자
};
```
Term클래스가 가지는 의미는 위에서 언급했듯이 쉽게 이해할 수 있을 것이다. 하나의 항 자체라고 생각하자.<br>
두번째는 Polynomial클래스에서 termarray라는 배열을 term클래스를 이용해서 private멤버변수로 가지고, 이 배열의
크기와 항의 수를 체크하기 위해서 capacity, terms라는 변수를 선언해주었다.<br>
한 칸(term)에 term은 coef와 exp를 가진다. 이 칸들의 연속이 termarray.
아래의 public멤버함수는 아래에서 순차적으로 설명하겠다.<br>


*생성자 함수*
```c++
Polynomial::Polynomial(void)
{
    capacity = 4;
    terms = 0 ;
    termArray = new Term[capacity] ;
}
```
클래스를 정의할 때는, 메모리가 할당되어지지 않고 main함수에서 이 클래스의 객체가 선언될때 생성자 함수가 실행되고
비로소 메모리가 할당된다.<br>
왜? capacity(배열의크기)는 4로 고정되었을까? 메모리낭비를 최소화하기 위해서이다. 사용자가 얼마나 많은 수의 항(term)을
입력할지 누구도 알지 못한다. 항의 수는 4가 될수도있고, 1000이 될수도있다. 프로그램을 더욱 최적화(메모리낭비를 줄인다.)하려면
capacity(배열의 크기)를 최소로 잡고 입력해주는 과정에서 이 배열의 크기를 오버한다면, 다시 배열의 크기를 넓혀주는것이 바람직하다.
그래서 나는 그 크기를 4로 지정하였다. terms는 당연히 아직 입력된 값이 없고 바로 생성직후니 항의 수가 0인 것은 당연하다.


*Print()*
```c++
void Polynomial::Print()
{
    int i ;
    cout << "\n" ;
    if (terms) {
        for (i = 0 ; i < terms-1 ; i++)
            cout << termArray[i].coef << "x^" << termArray[i].exp << " + " ;
        cout << termArray[i].coef << "x^" << termArray[i].exp << "\n" ;
    }
    else
        cout << " No terms " ;
    cout << "\n";
}
```
클래스의 각 객체의 polynomial을 출력한다.


*NewTerm*
```c++
void Polynomial::NewTerm(const float theCoeff, const int theExp){
    // 새로운 항을 termArray 끝에 첨가한다.
    if(terms == capacity) {
        //termArray의 크기를 두 배로 확장한다.
        capacity *= 2;
        Term *temp = new Term[capacity]; //새로운 배열
        for(int i = 0; i < terms; i++)
        {
            temp[i] = termArray[i];
        }
        delete [] termArray; //그 전 메모리를 반환한다.
        termArray = temp;
    }
    termArray[terms].coef = theCoeff;
    termArray[terms++].exp = theExp; // 차례로 terms가 증가한다.
    //term++ : ex) 0번째 항의 입력을 마치고 1로 증가시킨다(term++후위증가연산)
    // term = term + 1을 마지막에 해준것과 동일.
}
```
newterm의 이름에서 직관적으로 알 수 있듯이, 항을 입력하는 함수이다.
앞서, capacity의 초기값은 4로 설정해주었다. 하지만 입력된 항의수(terms)와 capacity(4)와 같은 시점에서
다시 메모리 확보를 위해 이를 두배증가 시켜줄 것 인데, 이미 만들어진 termArray의 메모리를 다시 재할당할 수 없으니,
temp라는 새로운 배열(기존 capacity의 값의 두배의 크기를 가짐)을 만들어서 기존의 termArray의 내용을 담아주고,
그 전에 담고있던(termarray의 메모리)는 delete로 밀어버린다. 그 후, 초기화된 termarray에 temp를 담아주면,
원하는 더 큰 집에 기존 데이터들의 내용을 담긴다.


*ADD*
```c++
Polynomial Polynomial::Add(Polynomial b){
    //a(x)(*this의 값)와 b(x)를 더한 결과를 반환한다.
    Polynomial c;
    int aPos = 0, bPos = 0;
    
    while((aPos < terms) && (bPos < b.terms)){
        if(termArray[aPos].exp == b.termArray[bPos].exp){
            float t = termArray[aPos].coef + b.termArray[bPos].coef;
            if(t) c.NewTerm(t, termArray[aPos].exp);
            aPos++;
            bPos++;
        } else if (termArray[aPos].exp < b.termArray[bPos].exp){
            c.NewTerm(b.termArray[bPos].coef, b.termArray[bPos].exp);
            bPos++;
        } else {
            c.NewTerm(termArray[aPos].coef, termArray[aPos].exp);
            aPos++;
        }
    }
    //end of while
    //A(x)의 나머지항들을 추가한다.
    for(; aPos < terms; aPos++)
        c.NewTerm(termArray[aPos].coef, termArray[aPos].exp);
    
    //B(x)의 나머지항들을 추가한다.
    for(; bPos < b.terms; bPos++)
        c.NewTerm(b.termArray[bPos].coef, b.termArray[bPos].exp);
    
    return c;
    
} 
```
newTerm함수에서 입력된 두 다항식을 더할 것 인데, Pos은 Position의 줄임말이다. 두 다항식을 더하기를 할때, 지수를 보고
동일한 지수의 계수를 더하고 없으면 기존의 지수와 계수를 유지하면서 더하는 것이 원칙이다. Pos이 0이라는 뜻은
첫번째 항의 위치를 의미하고 이를 증가하면서 다음 항들의 덧셈을 실행한다.
위 규칙을 따라, c에 이 a,b의 항들을 더해서 newTerm함수로 항을 생성한다.


*Multiply*
```c++
Polynomial Polynomial::Multiply(Polynomial b){
    Polynomial c,temp;
    int aPos = 0, bPos = 0;
    while(bPos < b.terms){
        for(; aPos < terms; aPos++){
            c.NewTerm((termArray[aPos].coef) * (b.termArray[bPos].coef), (termArray[aPos].exp) + (b.termArray[bPos].exp));
        }
        temp = temp.Add(c);
        aPos = 0;
        bPos++;
        if(bPos != b.terms){
            c.capacity = 4;
            c.terms = 0;
        }
    }
    return temp;
}
```
다항식의 곱셈을 손으로 한번 해보면, A다항식과 B다항식의 곱셉에 경우 B디항식의 첫째 항을 A다항식의 모든 항을 곱하여 C라는 다항식을 생성하고 또 다음 B다항식의 두번째항을 A다항식의 모든항과 곱하여 C'라는 식을 생성하고 C와 C'를 더하여 나아간다.


실행코드
===
>input<br>
>>다항식 A의 항의 수 : 5<br>
>>다항식 A의 1번째 항의 계수와 지수: 2 5<br>
>>다항식 A의 2번째 항의 계수와 지수: 2 4<br>
>>다항식 A의 3번째 항의 계수와 지수: 2 3<br>
>>다항식 A의 4번째 항의 계수와 지수: 2 2<br>
>>다항식 A의 5번째 항의 계수와 지수: 2 1<br>
>>print : 2x^5 + 2x^4 + 2x^3 + 2x^2 + 2x^1<br>
>>다항식의 B의 항의 수: 3<br>
>>다항식 B의1번째 항의 계수와 지수: 2 3<br>
>>다항식 B의2번째 항의 계수와 지수: 2 2<br>
>>다항식 B의3번째 항의 계수와 지수: 2 1<br>
>>print : 2x^3 + 2x^2 + 2x^1<br>

>output<br>
>>4x^8 + 8x^7 + 12x^6 + 12x^5 + 12x^4 + 8x^3 + 4x^2

모든코드
===
```c++
//
//  main.cpp
//  poly
//
//  Created by 이신육 on 22/09/2019.
//  Copyright © 2019 이신육. All rights reserved.
//

#include <iostream>
using namespace std;

class Polynomial; //전방선언
class Term {
    friend Polynomial; //Polynomial 클래스에서 접근이 가능하다.
private:
    float coef; // 계수
    int exp; // 지수
};

class Polynomial
{
private:
    Term *termArray;
    int capacity; // 배열크기
    int terms; // 0이 아닌 항의 수
public:
    void Print(void);
    void NewTerm(const float theCoeff, const int theExp);
    Polynomial Add(Polynomial b);
    Polynomial Multiply(Polynomial b);
    Polynomial(void); //생성자
};
Polynomial::Polynomial(void)
{
    capacity = 4;
    terms = 0 ;
    termArray = new Term[capacity] ;
}
Polynomial Polynomial::Add(Polynomial b){
    //a(x)(*this의 값)와 b(x)를 더한 결과를 반환한다.
    Polynomial c;
    int aPos = 0, bPos = 0;
    
    while((aPos < terms) && (bPos < b.terms)){
        if(termArray[aPos].exp == b.termArray[bPos].exp){
            float t = termArray[aPos].coef + b.termArray[bPos].coef;
            if(t) c.NewTerm(t, termArray[aPos].exp);
            aPos++;
            bPos++;
        } else if (termArray[aPos].exp < b.termArray[bPos].exp){
            c.NewTerm(b.termArray[bPos].coef, b.termArray[bPos].exp);
            bPos++;
        } else {
            c.NewTerm(termArray[aPos].coef, termArray[aPos].exp);
            aPos++;
        }
    }
    //end of while
    //A(x)의 나머지항들을 추가한다.
    for(; aPos < terms; aPos++)
        c.NewTerm(termArray[aPos].coef, termArray[aPos].exp);
    
    //B(x)의 나머지항들을 추가한다.
    for(; bPos < b.terms; bPos++)
        c.NewTerm(b.termArray[bPos].coef, b.termArray[bPos].exp);
    
    return c;
    
} // End of Add
Polynomial Polynomial::Multiply(Polynomial b){
    Polynomial c,temp;
    int aPos = 0, bPos = 0;
    while(bPos < b.terms){
        for(; aPos < terms; aPos++){
            c.NewTerm((termArray[aPos].coef) * (b.termArray[bPos].coef), (termArray[aPos].exp) + (b.termArray[bPos].exp));
        }
        temp = temp.Add(c);
        aPos = 0;
        bPos++;
        if(bPos != b.terms){
            c.capacity = 4;
            c.terms = 0;
        }
    }
    return temp;
}
void Polynomial::Print()
{
    int i ;
    cout << "\n" ;
    if (terms) {
        for (i = 0 ; i < terms-1 ; i++)
            cout << termArray[i].coef << "x^" << termArray[i].exp << " + " ;
        cout << termArray[i].coef << "x^" << termArray[i].exp << "\n" ;
    }
    else
        cout << " No terms " ;
    cout << "\n";
    
}
void Polynomial::NewTerm(const float theCoeff, const int theExp){
    // 새로운 항을 termArray 끝에 첨가한다.
    if(terms == capacity) {
        //termArray의 크기를 두 배로 확장한다.
        capacity *= 2;
        Term *temp = new Term[capacity]; //새로운 배열
        for(int i = 0; i < terms; i++)
        {
            temp[i] = termArray[i];
        }
        delete [] termArray; //그 전 메모리를 반환한다.
        termArray = temp;
    }
    termArray[terms].coef = theCoeff;
    termArray[terms++].exp = theExp; // 차례로 terms가 증가한다.
}

// main Function
int main(void) {
    Polynomial A, B, C;
    int i, n, e;
    float c;
    // n : 항의 수 , c : 항의 계수, e : 항의 지수
    
    
    cout << "다항식 A의 항의 수 : ";
    cin >> n;
    for(i = 1; i <= n; i++) {
        cout << "다항식 A의 " << i << "번째 항의 계수와 지수: ";
        cin >> c >> e;
        A.NewTerm(c, e);
    }
    A.Print();
    cout << "다항식의 B의 항의 수: ";
    cin >> n;
    for(i = 1; i <= n; i++){
        cout << "다항식 B의" << i << "번째 항의 계수와 지수: ";
        cin >> c >> e;
        B.NewTerm(c, e);
    }
    B.Print();
    
//    C = A.Add(B);
    C = A.Multiply(B);
    C.Print();
    
    return 0;
}

```

마무리
===
어떻게 프로그래밍하느냐에 따라 poly에 다양한 답이 있을 수 있고, 나는 가독성이 뛰어나고 효율적인 코드를 지향한다.
아직 나는 이런부분에 대해 미흡하여 남들의 코드를 한번 훑어볼 예정이다. 추후에 훨씬 보기 좋은 코드와 더 소개할 내용이 있으면 
위 코드의 시간복잡도와 함께 소개하겠다.






