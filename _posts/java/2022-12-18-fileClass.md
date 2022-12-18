---
title: "Java의 File 클래스"
layout: single
categories: java
typora-root-url: ..\..\images\
toc: true
---



# 1. File 클래스

- 파일 정보 제공, 파일 생성 및 삭제 기능 제공, 디렉토리 관리 기능을 제공하는 Java의 기본 클래스

## 1) 파일 객체 생성

```java
File file = new File("C:/Temp/file.txt");
File file = new File("C:\\Temp\\file.txt");
```

## 2) 메서드

| 순서 | 반환타입      | 메소드 이름                                                 | 설명                                                         |
| ---- | ------------- | ----------------------------------------------------------- | ------------------------------------------------------------ |
| 1    | boolean       | canExecute()                                                | 어플리케이션이 해당 파일을 실행할 수 있는지 테스한다.        |
| 2    | boolean       | canRead()                                                   | 어플리케이션이 해당 파일을 읽을 수 있는지 테스트한다.        |
| 3    | boolean       | canWrite()                                                  | 어플리케이션이 해당 파일을 변경할 수 있는지 테스트한다.      |
| 4    | int           | compareTo(File pathname)                                    | 파일경로를 사전식으로 비교한다. 결과(0 : 일치, 음수 : 파일경로가 사전식으로 작은경우, 양수 : 파일경로가 사전식으로 큰경우) ** 경로는 UNIX 시스템에서 대문자가 중요하다. WIN은 무관하다. |
| 5    | boolean       | createNewFile()                                             | 동일한 이름의 파일이 존재하지 않은 경우, 이 파일의 파일경로를 가지는 파일을 생성한다. |
| 6    | static File   | createTempFile(String prefix, String suffix)                | 지정된 임시 파일 디렉토리에 새 빈(empty) 파일을 생성한다. 파라미터 접두사와 접미사를 사용해 그 파일 이름을 생성한다. |
| 7    | static File   | createTempFile(String prefix, String suffix,File directory) | 파라미터 디렉토리에 새 빈(empty) 파일을 생성한다. 파라미터 접두사와 접미사를 사용해 그 파일 이름을 생성한다. |
| 8    | boolean       | delete()                                                    | 이 파일의 파일 경로에 파일 또는 디렉토리를 삭제한다.         |
| 9    | void          | deleteOnExit()                                              | 이 파일의 파일 경로에 파일 또는 디렉토리를 가상머신 종료 시 삭제요청한다. |
| 10   | boolean       | equals(Object obj)                                          | 이 파일의 파일 경로와 파라미터 객체가 동일한지 여부 확인.    |
| 11   | boolean       | exists()                                                    | 이 파일의 파일 경로의 파일 또는 디렉토리의 존재 여부 확인.   |
| 12   | File          | getAbsoluteFile()                                           | 이 파일의 파일 경로에 절대형을 반환한다.                     |
| 13   | String        | getAbsolutePath()                                           | 이 파일의 파일 경로 중 절대 경로를 문화열로 반환한다.        |
| 14   | File          | getCanonicalFile()                                          | 이 파일의 파일 경로에 정규 형식을 반환한다.                  |
| 15   | String        | getCanonicalPath()                                          | 이 파일의 파일 경로 중 정규 경로를 문자열로 반환한다.        |
| 16   | long          | getFreeSpace()                                              | 이 파일이 속한 파티션 내 미할당 크기(바이트)를 반환한다.     |
| 17   | String        | getName()                                                   | 이 파일의 파일이름 또는 디렉토리 이름을 반환한다.            |
| 18   | String        | getParent()                                                 | 이 파일의 파일경로의 부모 경로를 반환한다. 또는 디렉토리 이름이 아닌 경우 NULL. |
| 19   | File          | getParentFile()                                             | 이 파일의 파일 경로의 부모 경로를 반환한다. 또는 NULL        |
| 20   | String        | getPath()                                                   | 이 파일의 파일 경로를 문자열로 반환한다.                     |
| 21   | long          | getTotalSpace()                                             | 이 파일이 속한 파티션의 파티션 크기를 반환한다.              |
| 22   | long          | getUsableSpace()                                            | 이 파일이 속한 파티션의 사용가능한 바이트 수를 반환한다.     |
| 23   | int           | hashCode()                                                  | 이 파일의 파일경로를 해시코드로 계산한다.                    |
| 24   | boolean       | isAbsolute()                                                | 이 파일의 파일경로가 절대인지 여부 확인.                     |
| 25   | boolean       | isDirectory()                                               | 이 파일의 파일경로가 디렉토리(파일)인지 여부 확인.           |
| 26   | boolean       | isFile()                                                    | 이 파일의 파일경로가 파일(디렉토리)인지 여부 확인.           |
| 27   | boolean       | isHidden()                                                  | 이 파일의 숨김 속성인지 여부 확인.                           |
| 28   | long          | lastModified()                                              | 이 파일의 마지막 변경시각을 반환한다.                        |
| 29   | long          | length()                                                    | 이 파일의 파일 길이를 반환한다.                              |
| 30   | String[]      | list()                                                      | 이 파일의 파일경로에 디렉토리 내 파일 및 디렉토리를 나타내는 캐릭터 라인을 배열로 반환한다. |
| 31   | String[]      | list(FilenameFilter filter)                                 | 지정된 필터를 채우는 파일경로를 나타내는 디렉토리 내 파일 및 디렉토리를 나타내는 캐릭터 라인을 배열로 반환한다. |
| 32   | File[]        | listFiles()                                                 | 이 파일의 파일경로에 디렉토리 내 파일의 경로를 반환한다.     |
| 33   | File[]        | listFiles(FileFilter filter)                                | 이 파일의 파일경로에 디렉토리 내 파일과 디렉토리경로를 반환한다. |
| 34   | File[]        | listFiles(FilenameFilter filter)                            | 이 파일의 파일경로에 디렉토리 내 파일과 디렉토리경로를 반환한다. |
| 35   | static File[] | listRoots()                                                 | 사용가능한 파일 시스템 경로를 반환한다.                      |
| 36   | boolean       | mkdir()                                                     | 이 파일의 파일시스템 경로의 디렉토리를 생성한다.             |
| 37   | boolean       | mkdirs()                                                    | 이 파일의 파일시스템 경로의(존재하지 않은경우) 부모,하위 디렉토리 포함해 디렉토리를 생성한다. |
| 38   | boolean       | renameTo(File dest)                                         | 이 파일의 파일이름을 수정한다.                               |
| 39   | boolean       | setExecutable(boolean executable)                           | 이 파일의 실행 권한에 (소유자) 사용권한을 설정한다.          |
| 40   | boolean       | setExecutable(boolean executable, boolean ownerOnly)        | 이 파일의 실행 권한에 (소유자, 사용자) 사용권한을 설정한다.  |
| 41   | boolean       | setLastModified(long time)                                  | 이 파일의 파일 및 경로 디렉토리의 최종수정시각을 설정한다.   |
| 42   | boolean       | setReadable(boolean readable)                               | 이 파일의 읽기 권한에 (소유자) 사용권한을 설정한다.          |
| 43   | boolean       | setReadable(boolean readable, boolean ownerOnly)            | 이 파일의 읽기 권한에 (소유자, 사용자) 사용권한을 설정한다.  |
| 44   | boolean       | setReadOnly()                                               | 이 파일의 파읽 읽기권한 및 경로의 디렉토리 읽기권한을 읽기만 가능하도록 설정한다. |
| 45   | boolean       | setWritable(boolean writable)                               | 이 파일의 쓰기 권한에 (소유자) 사용권한을 설정한다.          |
| 46   | boolean       | setWritable(boolean writable, boolean ownerOnly)            | 이 파일의 쓰기 권한에 (소유자, 사용자) 사용권한을 설정한다.  |
| 47   | Path          | toPath()                                                    | 파일경로(추상패스)를 java.nio.file.Path 로 반환한다.         |
| 48   | String        | toString()                                                  | 파일경로(추상패스)를 문자열로 반환환다.                      |
| 49   | URL           | toURI()                                                     | 파일경로(추상패스)를 URI 로 구축하여 반환한다.               |
| 50   | URL           | toURL()                                                     | 지원되지 않음. toURI() 사용해 URI 만든 후 URL 로 변경하는 방법을 권장한다. |



# 2. File 입출력

## 1) FileInputStream

- 파일로부터 바이트 단위로 읽어 들일 때 사용
- 그림, 오디오, 비디오, 텍스트 파일 등 모든 종류의 파일을 읽을 수 있음

```java
// 객체 생성 방법 1
FileInputStream fis = new FileInputStream("C:/Temp/image.gif");
// 2
File file = new File("C:/Temp/image.gif");
FileInputStream fis = new FileInputStream(file);
```

- 파일이 존재하지 않으면 `FileNotFoundException`
- **`InputStream` 하위 클래스이므로 사용 방법이 `InputStream`과 동일!!**!

## 2) FileOutputStream

- 파일에 바이트 단위로 데이터를 저장할 때 사용
- 그림, 오디오, 비디오, 텍스트 파일 등 모든 종류의 파일을 저장할 수 있음

```java
// 객체 생성 방법 1
FileOutputStream fis = new FileOutputStream("C:/Temp/image.gif");
// 2
File file = new File("C:/Temp/image.gif");
FileOutputStream fis = new FileOutputStream(file);
```

- 파일이 이미 존재할 경우, 데이터 출력을 하게 되면 파일을 덮어 쓰게 됨.

- 끝에 데이터를 추가할 경우, 두 번째 매개변수를 `true`를 주면 됨.

  ```java
  FileOutputStream fis = new FileOutputStream(file, true);
  ```

- **`OutputStream` 하위 클래스이므로 사용 방법이 `OutputStream`과 동일!!**!

## 3) FileReader

- 텍스트 파일로부터 데이터를 읽어 들일 때 사용
- 문자 단위로 읽기 때문에 텍스트만 읽을 수 있음

```java
// 객체 생성 방법 1
FileReader fis = new FileReader("C:/Temp/image.gif");
// 2
File file = new File("C:/Temp/image.gif");
FileReader fis = new FileReader(file);
```

- **`Reader` 하위 클래스이므로 사용 방법이 `Reader`와 동일!!**

## 4) FileWriter

- 텍스트 파일에 문자 데이터를 저장할 때 사용
- 문자 단위로 쓰기 때문에 텍스트만 저장할 수 있음

```java
// 객체 생성 방법 1
FileWriter fis = new FileWriter("C:/Temp/image.gif");
// 2
File file = new File("C:/Temp/image.gif");
FileWriter fis = new FileWriter(file);
```

- 파일이 이미 존재할 경우, 덮어쓰게 됨.

- 기존 파일에 내용을 추가할 경우, 두 번째 매개변수에 `true`를 주면 됨.

  ```java
  FileWriter fis = new FileWriter(file, true);
  ```

---

> 마지막 수정일시: 2022-12-18 18:48