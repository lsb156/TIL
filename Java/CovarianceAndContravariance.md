# Covariance and contravariance

## 변성
`변성`이란 제너릭 프로그래밍을 하면서 상속에 관련되어 생기는 이슈가 생길때 접하곤합니다.
`변성`에 대해서 간단한 예시를 통해서 설명을 하자면 아래와 같이 쉽게 설명이 가능합니다.

`Integer`는 `Number`를 상속받아 만들어진 객체입니다.
그래서 `Interger`는 `Number`의 하위 타입이라고 할 수 있어 아래와 같은 코딩이 가능합니다.

``` java
public void test() {
    List<Number> list;
    list.add(Integer.valueOf(1));
}
```

하지만 `List<Integer>`는 `List<Number>`의 하위타입이 될 수 없습니다.
이러한 상황에서 Java나 Kotlin에서는 `type parameter`에 `타입 경계`를 명시하여 `Sub-Type`, `Super-Type`을 가능하게 해줍니다.
그걸 `변성`이라고 합니다.

|경계|bound|kotlin|java|
|:--|:--|:--|:--|
|상위 경계|Upper bound|Type<out T>|Type<? extends T>|
|하위 경계|Lower bound|Type<in T>|Type<? super T>|

변성에 대한 의미는 아래와 같이 간결하게 설명이 가능합니다.

||의미|
|:--|:--|
|공변성(covariant)|T’가 T의 서브타입이면, `C<T’>`는 `C<T>`의 서브타입이다.|
|반공변성(contravariant)|T’가 T의 서브타입이면, `C<T>`는 `C<T’>`의 서브타입이다.|
|무공변성(invariant)|C<T>와 C<T’>는 아무 관계가 없다.|

위에서 설명한것과 같이 타입경계를 지정하여 적용이된 Kotlin과 Java에 대한 설명은 아래와 같이 해석이 가능합니다.

||Kotlin|Java|
|:--|:--|
|공변성(covariant)|T’가 T의 서브타입이면, `C<T’>`는 `C<out T>`의 서브타입이다.|T’가 T의 서브타입이면, `C<T’>`는 `C<? extends T>`의 서브타입이다.|
|반공변성(contravariant)|T’가 T의 서브타입이면, `C<T>`는 `C<in T’>`의 서브타입이다.|T’가 T의 서브타입이면, `C<T>`는 `C<? super T’>`의 서브타입이다.|
|무공변성(invariant)|C<T>와 C<T’>는 아무 관계가 없다.|C<T>와 C<T’>는 아무 관계가 없다.|

### 공변성 (Covariant)
공변성은 타입생성자에게 리스코프 치환 법칙을 허용하여 유연한 설계를 가능하게 해줍니다.

``` kotlin
interface Cage<T> {
    fun get(): T
}

open class Animal

open class Hamster(var name: String) : Animal()

class GoldenHamster(name: String) : Hamster(name)

fun tamingHamster(cage: Cage<out Hamster>) {
    println("길들이기 : ${cage.get().name}")
}

fun main() {

    val animal = object : Cage<Animal> {
        override fun get(): Animal {
            return Animal()
        }
    }
    val hamster = object : Cage<Hamster> {
        override fun get(): Hamster {
            return Hamster("Hamster")
        }
    }
    val goldenHamster = object : Cage<GoldenHamster> {
        override fun get(): GoldenHamster {
            return GoldenHamster("Leo")
        }
    }

    tamingHamster(animal) // compile Error
    tamingHamster(hamster)
    tamingHamster(goldenHamster)
}
```
tamingHamster 함수는 Hamster의 서브타입만을 받기때문에 animal 변수는 들어갈 수 없습니다.


### 반공변성 (Contravariant)
공변성의 반대 개념으로 자기 자신과 부모 객체만을 허용합니다.
``` kotlin
interface Cage<T> {
    fun get(): T
}

open class Animal

open class Hamster(var name: String) : Animal()

class GoldenHamster(name: String) : Hamster(name)

fun ancestorOfHamster(cage: Cage<in Hamster>) {
    println("ancestor = ${cage.get()::javaClass.name}")
}

fun main() {

    val animal = object : Cage<Animal> {
        override fun get(): Animal {
            return Animal()
        }
    }
    val hamster = object : Cage<Hamster> {
        override fun get(): Hamster {
            return Hamster("Hamster")
        }
    }
    val goldenHamster = object : Cage<GoldenHamster> {
        override fun get(): GoldenHamster {
            return GoldenHamster("Leo")
        }
    }

    ancestorOfHamster(animal) 
    ancestorOfHamster(hamster)
    ancestorOfHamster(goldenHamster) // compile Error
}
```
`ancestorOfHamster`에서 햄스터의 조상을 찾는 함수를 구현하여 햄스터를 포함한 그 조성들만 허용하도록 제한하였습니다.
하위타입인 `Cage<GoldenHamster>`는 제한에 걸려있어 compile error가 나는것을 확인 할 수 있습니다.

### 무공변성 (invariant)
Java, Kotlin의 `Generic`은 기본적으로 무공변성으로 아무런 설정이 없는 기본 `Generic`을 말합니다.
``` kotlin
interface Cage<T> {
    fun get(): T
}

open class Animal

open class Hamster(var name: String) : Animal()

class GoldenHamster(name: String) : Hamster(name)

fun matingGoldenHamster(cage: Cage<GoldenHamster>) {
    val hamster = GoldenHamster("stew")
    println("교배 : ${hamster.name} & ${cage.get().name}")
}

fun main() {

    val animal = object : Cage<Animal> {
        override fun get(): Animal {
            return Animal()
        }
    }
    val hamster = object : Cage<Hamster> {
        override fun get(): Hamster {
            return Hamster("Hamster")
        }
    }
    val goldenHamster = object : Cage<GoldenHamster> {
        override fun get(): GoldenHamster {
            return GoldenHamster("Leo")
        }
    }

    matingGoldenHamster(animal) // compile Error
    matingGoldenHamster(hamster) // compile Error
    matingGoldenHamster(goldenHamster)
}
```
위 코드의 `Cage<Animal>`, `Cage<Hamster>`, `Cage<GoldenHamster>`는 서로 각각 연관이 없는 객체로서 무공변성의 적절한 예입니다.

## 지점에 따른 변성
변성에 따른 타입을 나누는 것에 대해서 지점에 따른 변성으로 선언 지점 변성과 사용 지점 변성으로 나뉠 수 있습니다.

### 선언 지점 변성
클래스를 선언하면서 클래스 자체에 변성을 지정하는 방식(클래스에 in/out을 지정하는 방식)을 선언 지점 변성(declaration-site variance)이라고 합니다.
선언 하면서 지정하면, 클래스의 공변성을 전체적으로 지정하는게 되기 때문에 클래스를 사용하는 장소에서는 따로 타입을 지정해줄 필요가 없어 편리하게 됩니다.

### 사용 지점 변성
사용 지점 변성(use-site variance)은 메소드 파라미터에서, 또는 제네릭 클래스를 생성할 때 등 구체적인 사용 위치에서 변성을 지정하는 방식입니다.
Java에서 사용하는 한정 와일드카드(bounded wildcard)가 바로 이 방식입니다.
이는 타입 파라미터가 있는 타입을 사용할 때마다 해당 타입 파라미터를 하위 타입이나 상위 타입 중 어떤 타입으로 대치할 수 있는지를 명시해야 합니다.
