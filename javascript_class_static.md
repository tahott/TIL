# JavaScript class의 static에 관하여 정리
예전 어느 면접에서 코드에 static을 사용했는데 static의 역할이 무엇인지에 대해<br>
제대로 답변하지 못한 나를 반성하며 정리한다.

## static fields
- static fields는 캐시, 고정된 설정값, 인스턴스 간 복제가 필요 없는 데이터에 사용
- 인스턴스화 전에 사용이 가능하며 인스턴스화 후에는 사용할 수 없다
  ```ts
  class SomeClass {
    static SOME_STRING = "talk is cheap, show me the code";
  }

  console.log(SomeClass.SOME_STRING); // "talk is cheap, show me the code"
    
  const someClass = new SomeClass();

  console.log(someClass.SOME_STRING); // undefined
  ```
- 인스턴스끼리의 static fileds 값은 공유 된다
  ```ts
  class Book {
    static #MAX_BOOK_INSTANCE = 2;
    static #bookInstance = 0;

    #name;

    constructor(name: string) {
        Book.#bookInstance += 1;

        if (Book.#bookInstance > Book.#MAX_BOOK_INSTANCE) {
            throw new Error("Unable to create Book instance");
        }
        this.#name = name;
    }

    get name() {
        return this.#name;
    }

    set name(name: string) {
        this.#name = name;
    }
  }

  new Book("The Pragmatic Programmer");
  new Book("Clean Architecture");
  new Book("The little prince"); // Error: "Unable to create Book instance"

  console.log(Book.#bookInstance); // Error: "Private field '#instance' must be declared in an enclosing class"
  ```

## static method
- static method는 class의 instance가 아닌 class에 관련한 논리를 보유
- static method는 static fields에 access 가능
- static method는 instance fileds에 access 불가능
  ```ts
  class Library {
      static #bookList: Array<string> = [];

      libraryName;
      constructor(name: string) {
          this.libraryName = name;
      }

      addBook(book: string) {
          Library.#bookList.push(book);
      }

      static getBookList() {
          console.log(this.libraryName);
          console.log(Library.#bookList);
      }

      callLibraryName() {
          console.log(this.libraryName);
      }
  }

  const library = new Library("National Library of Korea");

  library.addBook("The little prince");
  library.addBook("The little mermaid");

  Library.getBookList(); // undefined, ["The little prince", "The little mermaid"]

  library.callLibraryName(); // "National Library of Korea"
  ```