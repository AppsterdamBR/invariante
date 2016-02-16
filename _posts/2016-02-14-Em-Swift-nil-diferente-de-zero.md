---
layout: post
title: Em Swift, nil é diferente de Zero
---

`Objective-C` possui uma característica muito interessante e polêmica, enviar mensagens para `nil` é válido e não lança excessão como em outras linguagens. Não vou discutir se isso é bom ou ruim, mas é algo que gosto e uso frequentemente quando programo nessa linguagem.

Um uso comum seria quando é necessário testar se um *array* é vazio:

~~~ objc
if(array.count == 0) {
    NSLog(@"empty array");
}
~~~

Isso funciona porque o retorno de qualquer mensagem enviada para `nil` é `0`, `nil` ou `NULL` dependendo do tipo de retorno da mensagem, semanticamente diferentes mas tecnicamente iguais a `ZERO`. 

Então se `array` for `nil`, `array.count` retorna `0`, mais uma vitória do bem e menos código escrito 😎.

Sem pensar muito o podemos escrever o equivalente em `Swift`:

~~~ swift
if array.count == 0 {
    print("empty array")
}
~~~

Se `array` for um opcional, novamente sem pensar muito, poderíamos fazer um *optional chaining* e só colocar um `?`:

~~~ swift
if array?.count == 0 {
    print("empty array")
}
~~~

__NÃO!!__ Quando `array = nil`, `array?.count == 0` é falso, mas porque?

A resposta está em no funcionamento do [*optional chaining*](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/OptionalChaining.html): `?` tem um comportamento semelhante ao `!` (*force unwrapping*), com a diferença que se o valor opcional for `nil` não é lançado um erro de *runtime*. Como não ocorre a interrupção do programa e afim de evitar inconsistências o resultado de chamadas de métodos, propriedades e *subscripts* sempre vão retornar um opcional, mesmo que o tipo original não seja. Por exemplo:

~~~ swift
var array: [Int]?
let count = array?.count // count is Int? not Int
~~~

Isso significa que quando `array = nil`, `count = nil` que não é igual a `0` e por isso o teste anterior falha! Ou seja em `Swift`, `nil != 0`!

Para escrever o teste de maneira que funciona temos várias opções.

Definir que quando o resultado do `count` for `nil` o resultado esperado é `0`:

~~~ swift
if (array?.count ?? 0) == 0 {
    print("empty array")
}
~~~

Testar o `nil` e fazer *force unwrapping*. Não me agrada o *force unwrapping*, sei que nesse caso nunca aconteceria um erro de *runtime* mas prefiro evitar ao máximo o `!`:

~~~ swift
if array == nil || array!.count == 0 {
    print("empty array")
}
~~~

Por algum motivo ainda desconhecido para mim, `nil` é menor que qualquer `Int`. Então temos uma opção que eu não recomendo, ¯\\\_(ツ)\_/¯:

~~~ swift
if !(array?.count > 0) {
    print("empty array")
}
~~~

Com certeza devem haver mais uma dezena de maneiras de escrever mas acho que já deu para ter uma idéia. Provavelmente o erro desse caso é transpor exatamente a mesma lógica do `Objective-C` para `Swift`.

---

*Update 2016/02/16* - O [Fabri](https://twitter.com/marcelofabri_) e o [Koga](https://twitter.com/brunokoga) lembraram que o `Array` adota o protocolo `CollectionType` e portanto o `isEmpty` seria mais adequado:

~~~ swift
if array?.isEmpty ?? true {
    print("empty array")
}
~~~

O Fabri pensou mais no assunto[^1] e propôs uma extensão para o `Optional` de `IntegerType` que evitaria o problema apresentado:

~~~ swift
extension Optional where Wrapped: IntegerType {
    var valueOrZero: Wrapped {
        return self ?? 0
    }
}
~~~

Assim o teste ficaria:

~~~ swift
if (array?.count).valueOrZero == 0 {
    print("empty")
}
~~~

Acho que a versão com o `isEmpty` ainda fica mais legível.

[^1]: Assumindo que eu seja um cara teimoso e me recuse a usar o `isEmpty`

---

Criticas, sugestões e comentários são sempre bem vindos, é só me *pingar* no [@diogot](https://twitter.com/diogot) ou no [slack do iOS Dev BR](http://iosdevbr.herokuapp.com).

---
Diogo Tridapalli <br />
[@diogot](https://twitter.com/diogot)
