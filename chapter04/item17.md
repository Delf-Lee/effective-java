# 아이템 17. 변경 가능성을 최소화하라 
 
 ## 클래스를 불변으로 만들기 위한 다섯 가지 규칙
 1. 객체의 상태를 변경하는 메서드(변경자)를 제공하지 않는다.
 2. 클래스를 확장할 수 없도록 한다.
 3. 모든 필드를 `final`로 선언한다.
 4. 모든 필드를 `private`로 선언한다.
 5. 자신 외에는 내부의 가변컴포넌트에 접근할 수 없도록 한다.


[Complex.java](/src/effectivejava/chapter4/item17/Complex.java)
``` java
package effectivejava.chapter4.item17;

// 코드 17-1 불변 복소수 클래스 (106-107쪽)
public final class Complex {
    private final double re;
    private final double im;

    public static final Complex ZERO = new Complex(0, 0);
    public static final Complex ONE  = new Complex(1, 0);
    public static final Complex I    = new Complex(0, 1);

    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public double realPart()      { return re; }
    public double imaginaryPart() { return im; }

    public Complex plus(Complex c) {
        return new Complex(re + c.re, im + c.im);
    }

    // 코드 17-2 정적 팩터리(private 생성자와 함께 사용해야 한다.) (110-111쪽)
    public static Complex valueOf(double re, double im) {
        return new Complex(re, im);
    }

    public Complex minus(Complex c) {
        return new Complex(re - c.re, im - c.im);
    }

    public Complex times(Complex c) {
        return new Complex(re * c.re - im * c.im,
                re * c.im + im * c.re);
    }

    public Complex dividedBy(Complex c) {
        double tmp = c.re * c.re + c.im * c.im;
        return new Complex((re * c.re + im * c.im) / tmp,
                (im * c.re - re * c.im) / tmp);
    }

    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof Complex))
            return false;
        Complex c = (Complex) o;

        // == 대신 compare를 사용하는 이유는 63쪽을 확인하라.
        return Double.compare(c.re, re) == 0
                && Double.compare(c.im, im) == 0;
    }
    @Override public int hashCode() {
        return 31 * Double.hashCode(re) + Double.hashCode(im);
    }

    @Override public String toString() {
        return "(" + re + " + " + im + "i)";
    }
}
```

## 예제에서 눈여겨 볼 점
- 필드는 `private final`로 선언되어 있다.
- 사칙연산을 수행하는 메서드가 있고 자신을 수정하지 않고 **새로운 `Complex` 인스턴스**를 반환한다.


이처럼 피연산자에 함수를 적용해 그 결과를 반환하지만, 피연산자 자체는 그대로인 프로그래밍 패턴을 **함수형 프로그래밍**이라고 한다.

## 함수형 프로그래밍
익숙하지 않다면 부자연스러워 보일 수 있지만, 이 방식으로 프로그래망 하면 코드에 불변이 되는 영역의 비율이 높아지는 장점을 누릴 수 있다.

## 불변 객체
### 특징
- **불변 객체는 생성된 시점의 상테를 파괴될 때 까지 그대로 간직한다.**
- **불변 객체는 thread-safe하며 동기화할 필요가 없고, 안심하고 공유할 수 있다.**
  - 이런 특성 때문에 한번 만든 불변 인스턴스를 최대한 재활용하는 것을 권장한다. (ex. 상수`public static final`)
  - 자주 사용되는 인스턴스를 캐싱하여 인스턴스를 중복 생성하지 않게 해주는 정적 팩터리를 제공할 수 있다.
  - 자유롭게 공유할 수 있다는 점은 *방어적 복사*도 필요없다는 말과 같다.
- **불변 객체끼리는 내부 데이터를 공유할 수 있다.**
  - 예로, `BigInteger`클래스에서는 부호(int)와 크기(int 배열)를 따로 표현하는데, 필요하다면 원본 인스턴스의 값을 반환한다. 그 결과 새로 만든 `BigInteger` 인스턴스도 원본 인스턴스가 가르키는 내부 배열을 그대로 가르킨다.
- **객체를 만들 때 다른 불변 객체들을 구성요소로 사용하면 이점이 많다.**
  - 구조가 아무리 복잡하더라고 불변식을 유지하기 훨씬 수월하기 때문이다.
  - 좋은 예로 맵의 키와 집합의 원소로 사용되는 객체
- **불변 객체는 그 자체로 실패 원자성을 제공한다.**
### 단점
단점은 값이 다르면 반드시독립된 객체로 만들어애 한다는 것이다. 이는 해당 객체가 크면 클수록 많은 비용을 필요로 한다. 그 대처 방법으로는
- 흔히 쓰일 다단계 연산(multistep operation)들을 예측하여 기본 기능으로 제공하는 것.
- 클래스를 `public`으로 제공하는 것

## 불변 클래스 보장
클래스가 불변임을 보장하려면 자신을 상속하지 못하게 해야한다. 자산을 상속하지 못하게 하는 가장 쉬운 방법은 `final` 클래스로 선언하는 것이지만, 더 유연한 방법이 있다. 
> 모든 생성자를 `private` 혹은 `package-private`로 만들고 `public` 정적 팩터리를 제공한다.
``` java
public class Complex {
    private final double re;
    private final double im;

    private Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public static Complex valueOf(double re, double im) {
        return new Complex(re, im);
    }
}
```
보통의 경우 이 방식이 최선일 때가 많다. 이렇게 정적 팩터리 방식을 사용하면,
- 접근 가능한 생성지가 없어서 클래스 확장이 불가능하다.
- 다수의 구현 클래스를 활용한 유연성을 제공하고, 이에 더해 다음 릴리스에서 객체 캐싱 기능을 추가해 성능을 끌어올릴 수도 있다.