# 아이템 9. try-finally보다는 try-with-resources를 사용하라
- 자바 라이브러리에서는 `close`메서드를 통해 직접 닫아줘야 하는 자원이 많다.
- 이런 자원 중 사당수가 `finalizer`를 활용하고 있지만, 그렇게 믿을만하지 못하다.

## try-finally
- 전통적으로 자원이 제대로 닫힘을 보장하는 수단으로는 try-finally가 쓰였다.
    - 하지만 자원을 여러개 사용한다면 코드가 지저분해지고 가독성이 떨어진다.
``` java
public class Copy {
    private static final int BUFFER_SIZE = 8 * 1024;

    // 코드 9-2 자원이 둘 이상이면 try-finally 방식은 너무 지저분하다! (47쪽)
    static void copy(String src, String dst) throws IOException {
        InputStream in = new FileInputStream(src);
        try {
            OutputStream out = new FileOutputStream(dst);
            try {
                byte[] buf = new byte[BUFFER_SIZE];
                int n;
                while ((n = in.read(buf)) >= 0)
                    out.write(buf, 0, n);
            } finally {
                out.close();
            }
        } finally {
            in.close();
        }
    }
}
```
- 이러할 경우 두 번째 예외가 첫 번째 예외를 완전히 집어삼키는 상황도 발생할 수 있다.
  - ex) 기기에 물리적 문제가 생겨 `readLine`메서드와 `close`메서드 모두 실패하는 경우.
  - 이런 경우에는 스택 추적 내역에 첫 번쨰 예외에 관한 정보는 남지 않게 되어, 실게 세스템에서 디버깅을 어렵게한다. 


## try-with-resource
이런 문제는 자바 7에서 나온 try-with-resource 덕에 모두 해결 되었다.
- 이 구조를 사용하려면 해당 자원이 `AutoCloseable` 인터페이스를 구현해야 한다.
``` java
public class Copy {
    private static final int BUFFER_SIZE = 8 * 1024;

    // 코드 9-4 복수의 자원을 처리하는 try-with-resources - 짧고 매혹적이다! (49쪽)
    static void copy(String src, String dst) throws IOException {
        try (InputStream   in = new FileInputStream(src);
             OutputStream out = new FileOutputStream(dst)) {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        }
    }
}
```
훨씬 짧고 읽기 수월하고 문제를 진단하기도 용이하다.
- 예외가 발생하면 프로그래머에게 보여줄 예외는 하나만 보존되고 여러 개의 다른 예외는 숨겨질 수 있다.

## 참조한 아이템
- 아이템8