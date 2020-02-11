# Declaração de variáveis

`let` e `const` são dois tipos relativamente novos de declarações de variáveis em JavaScript. [Como mencionamos anteriormente](./Basic%20Types.md#a-note-about-let), `let` é semelhante ao `var` em alguns aspectos, mas permite que os usuários evitem algumas das “pegadinhas” comuns que os usuários encontram no JavaScript. `const` é um adjunto de `let` e que impede a reatribuição a uma variável.

Com o TypeScript sendo um superconjunto de JavaScript, a linguagem naturalmente suporta `let` e `const`. 
Aqui, elaboraremos mais sobre essas novas declarações e por que elas são preferíveis a `var`.

Se você usou o JavaScript de forma não autorizada, a próxima seção pode ser uma boa maneira de atualizar sua memória.
Se você estiver intimamente familiarizado com todas as peculiaridades das declarações `var` no JavaScript, poderá achar mais fácil e seguir adiante.

# declaração `var`

Declaring a variable in JavaScript has always traditionally been done with the `var` keyword.
A declaração de uma variável em JavaScript é tradicionalmente feita com a palavra-chave `var`.

```ts
var a = 10;
```

Como você já deve ter percebido, acabamos de declarar uma variável chamada `a` com o valor` 10`.

Também podemos declarar uma variável dentro de uma função:

```ts
function f() {
    var messagem = "Olá, Mundo!";

    return messagem;
}
```

e também podemos acessar essas mesmas variáveis em outras funções:

```ts
function f() {
    var a = 10;
    return function g() {
        var b = a + 1;
        return b;
    }
}

var g = f();
g(); // returns '11'
```

Neste exemplo acima, `g` capturou a variável `a` declarada em `f`.
Em qualquer momento em que `g` for chamado, o valor de `a` será vinculado ao valor de `a` em `f`.
Mesmo se `g` for chamado assim que o `f` terminar de rodar, ele poderá acessar e modificar o `a`.

```ts
function f() {
    var a = 1;

    a = 2;
    var b = g();
    a = 3;

    return b;

    function g() {
        return a;
    }
}

f(); // returns '2'
```

## Regras de escopo

As exigências `var` têm algumas regras de escopo estranhas comparadas àquelas usadas em outras linguagens.
Veja o seguinte exemplo:

```ts
function f(shouldInitialize: boolean) {
    if (shouldInitialize) {
        var x = 10;
    }

    return x;
}

f(true);  // returns '10'
f(false); // returns 'undefined'
```

Alguns leitores podem dar uma olhada neste exemplo.
A variável `x` foi declarada *dentro do bloco `if`*, e ainda assim conseguimos acessá-la de fora desse bloco.
Isso ocorre porque as declarações `var` estão acessíveis em qualquer lugar dentro de sua função, módulo, _namespace_ ou escopo global - todos sobre os quais falaremos mais adiante - independentemente do bloco que o contenha.
Algumas pessoas chamam isso de *escopo-`var`* ou *escopo-de-função*.
Os parâmetros também têm escopo de função.


Essas regras de escopo podem causar vários tipos de erros.
Um problema que eles exacerbam é o fato de não ser um erro declarar a mesma variável diversas vezes:

```ts
function sumarMatriz(matriz: number[][]) {
    var soma = 0;
    for (var i = 0; i < matriz.length; i++) {
        var linhaAtual = matriz[i];
        for (var i = 0; i < linhaAtual.length; i++) {
            soma += linhaAtual[i];
        }
    }

    return soma;
}
```

Talvez fosse fácil identificar alguns, mas o loop interno `for` substituirá acidentalmente a variável `i` porque `i` se refere à mesma variável com escopo de função.
Como os desenvolvedores experientes sabem até agora, tipos semelhantes de bugs passam por revisões de código e podem ser uma fonte infinita de frustração.

## Peculiaridades de capitura de variável

Reserve um segundo rápido para adivinhar qual é a saída do seguinte snippet:

```ts
for (var i = 0; i < 10; i++) {
    setTimeout(function() { console.log(i); }, 100 * i);
}
```

Para quem não está familiarizado, o `setTimeout` tentará executar uma função após um certo número de milissegundos (apesar de esperar que qualquer outra coisa pare de executar).

Pronto? Dê uma olhada:

```text
10
10
10
10
10
10
10
10
10
10
```

Muitos desenvolvedores de JavaScript estão intimamente familiarizados com esse comportamento, mas se você for surpreendido, certamente não estará sozinho.
A maioria das pessoas espera que a saída seja

```text
0
1
2
3
4
5
6
7
8
9
```

Lembra do que mencionamos anteriormente sobre captura de variáveis?
Toda expressão de função que passamos para `setTimeout` na verdade se refere ao mesmo `i` do mesmo escopo.

Vamos levar um minuto para considerar o que isso significa.
`setTimeout` executará uma função após um número de milissegundos, *mas somente* após o loop `for` parar de executar;
No momento em que o loop `for` parou de executar, o valor de `i` é `10`.
Portanto, toda vez que a função especificada for chamada, ela imprimirá `10`!

Uma solução comum é usar um IIFE - uma expressão de função chamada imediatamente - para capturar `i` a cada iteração:

```ts
for (var i = 0; i < 10; i++) {
    // capture the current state of 'i'
    // by invoking a function with its current value
    (function(i) {
        setTimeout(function() { console.log(i); }, 100 * i);
    })(i);
}
```

Esse padrão de aparência estranha é realmente bastante comum.
O `i` na lista de parâmetros, na verdade, sombreia o `i` declarado no loop `for`, mas como os nomeamos da mesma forma, não precisamos modificar muito o corpo do loop.

# declaração do `let`

Até agora você descobriu que o `var` tem alguns problemas, e é exatamente por isso que as instruções `let` foram introduzidas.
Além da palavra-chave usada, as instruções `let` são escritas da mesma maneira que as instruções `var`.

```ts
let ola = "Olá!";
```

A principal diferença não está na sintaxe, mas na semântica, na qual abordaremos agora.

## Escopo de bloco

Quando uma variável é declarada usando `let`, ela usa o que alguns chamam de *escopo lexical* ou *escopo de bloco*.
Diferente das variáveis declaradas com `var`, cujos escopos vazam para a função que os contêm, as variáveis com escopo de bloco não são visíveis fora do bloco contendo mais próximo ou do loop `for`.

```ts
function f(input: boolean) {
    let a = 100;

    if (input) {
        // Still okay to reference 'a'
        let b = a + 1;
        return b;
    }

    // Error: 'b' doesn't exist here
    return b;
}
```

Aqui, temos duas variáveis locais `a` e `b`.
O escopo de `a` é limitado ao corpo de `f` enquanto o escopo de `b` é limitado ao bloco de instrução `if` que contém.

Variáveis declaradas em uma cláusula `catch 'também possuem regras de escopo semelhantes.

```ts
try {
    throw "oh no!";
}
catch (e) {
    console.log("Oh well.");
}

// Error: 'e' doesn't exist here
console.log(e);
```

Outra propriedade das variáveis com escopo em bloco é que elas não podem ser lidas ou gravadas antes de serem realmente declaradas.
Embora essas variáveis estejam "presentes" em todo o seu escopo, todos os pontos até sua declaração fazem parte de sua *zona morta temporal*.
Esta é apenas uma maneira sofisticada de dizer que você não pode acessá-los antes da declaração `let`, e felizmente o TypeScript informará isso.

```ts
a++; // illegal to use 'a' before it's declared;
let a;
```

Algo a ser observado é que você ainda pode *capturar* uma variável com escopo de bloco antes de ser declarada.
O único problema é que é ilegal chamar essa função antes da declaração.
Se estiver direcionado ao ES2015, um moderno _runtime_ gerará um erro; no entanto, no momento, o TypeScript é permissivo e não relatará isso como um erro.

```ts
function foo() {
    // okay to capture 'a'
    return a;
}

// illegal call 'foo' before 'a' is declared
// runtimes should throw an error here
foo();

let a;
```

Para mais informações sobre zonas mortas temporais, consulte o conteúdo relevante na [Mozilla Developer Network](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Statements/let#Temporal_dead_zone_and_errors_with_let).

## Re-declarações e Sombreamento(Shadowing)

Com declarações `var`, mencionamos que não importava quantas vezes você declarou suas variáveis; você acabou de pegar um.

```ts
function f(x) {
    var x;
    var x;

    if (true) {
        var x;
    }
}
```

No exemplo acima, todas as declarações de `x` realmente se referem ao *mesmo* `x`, e isso é perfeitamente válido.
Isso geralmente acaba sendo uma fonte de bugs.
Felizmente, as declarações `let` não são tão perdoadoras.

```ts
let x = 10;
let x = 20; // error: can't re-declare 'x' in the same scope
```

As variáveis não precisam necessariamente ter um escopo de bloco para o TypeScript nos dizer que há um problema.

```ts
function f(x) {
    let x = 100; // error: interferes with parameter declaration
}

function g() {
    let x = 100;
    var x = 100; // error: can't have both declarations of 'x'
}
```

Isso não quer dizer que a variável com escopo de bloco nunca possa ser declarada com uma variável com escopo de função.
A variável com escopo de bloco só precisa ser declarada dentro de um bloco distintamente diferente.

```ts
function f(condicao, x) {
    if (condicao) {
        let x = 100;
        return x;
    }

    return x;
}

f(false, 0); // returns '0'
f(true, 0);  // returns '100'
```

O ato de introduzir um novo nome em um escopo mais aninhado é chamado *sobreamento*(_shadowing_).
Isso é um pouco de uma faca de dois gumes, pois pode introduzir certos bugs por conta própria em caso de sombreamento acidental, além de impedir certos bugs.
Por exemplo, imagine que tivéssemos escrito nossa função `somaMatriz` anterior usando variáveis `let`.

```ts
function sumaMatriz(matriz: number[][]) {
    let soma = 0;
    for (let i = 0; i < matriz.length; i++) {
        var linhaAtual = matriz[i];
        for (let i = 0; i < linhaAtual.length; i++) {
            soma += linhaAtual[i];
        }
    }

    return soma;
}
```

Esta versão do loop realmente executará a soma corretamente, porque o `i` do loop interno sombreia o `i` do loop externo.
O sombreamento(_shadowing_) *normalmente* deve ser evitado no interesse de escrever um código mais claro.
Embora existam alguns cenários em que possa ser adequado tirar proveito disso, você deve usar seu bom senso.

## Capitura de variável em escopo de bloco

Quando tocamos pela primeira vez na ideia de captura de variáveis com a declaração `var`, analisamos brevemente como as variáveis agem uma vez capturadas.
Para dar uma melhor intuição disso, sempre que um escopo é executado, ele cria um "ambiente" de variáveis.
Esse ambiente e suas variáveis capturadas podem existir mesmo após a conclusão de tudo dentro de seu escopo.

```ts
function aCidadeQueSempreDorme() {
    let getCidade;

    if (true) {
        let cidade = "Seattle";
        getCidade = function() {
            return cidade;
        }
    }

    return getCidade();
}
```

Como capturamos `cidade` de dentro de seu ambiente, ainda podemos acessá-lo, apesar do bloco `if` ter terminado de executar.

Lembre-se de que, com o exemplo anterior `setTimeout`, acabamos precisando usar um IIFE para capturar o estado de uma variável para cada iteração do loop `for`.
Com efeito, o que estávamos fazendo era criar uma nova variável de ambiente para os nossos variáveis capturadas.
Isso foi um pouco trabalhoso, mas, felizmente, você nunca precisará fazer isso novamente no TypeScript.

As declarações `let 'têm comportamento drasticamente diferente quando declaradas como parte de um loop.
Em vez de apenas introduzir um novo ambiente no próprio loop, essas declarações meio que criam um novo escopo *por iteração*.
Como isso é o que estávamos fazendo de qualquer maneira com nosso IIFE, podemos mudar nosso antigo exemplo `setTimeout` para usar apenas uma declaração `let`.

```ts
for (let i = 0; i < 10 ; i++) {
    setTimeout(function() { console.log(i); }, 100 * i);
}
```

e como esperado, isso será impresso

```text
0
1
2
3
4
5
6
7
8
9
```

# `const` declarations

`const` declarations are another way of declaring variables.

```ts
const numLivesForCat = 9;
```

They are like `let` declarations but, as their name implies, their value cannot be changed once they are bound.
In other words, they have the same scoping rules as `let`, but you can't re-assign to them.

This should not be confused with the idea that the values they refer to are *immutable*.

```ts
const numLivesForCat = 9;
const kitty = {
    name: "Aurora",
    numLives: numLivesForCat,
}

// Error
kitty = {
    name: "Danielle",
    numLives: numLivesForCat
};

// all "okay"
kitty.name = "Rory";
kitty.name = "Kitty";
kitty.name = "Cat";
kitty.numLives--;
```

Unless you take specific measures to avoid it, the internal state of a `const` variable is still modifiable.
Fortunately, TypeScript allows you to specify that members of an object are `readonly`.
The [chapter on Interfaces](./Interfaces.md) has the details.

# `let` vs. `const`

Given that we have two types of declarations with similar scoping semantics, it's natural to find ourselves asking which one to use.
Like most broad questions, the answer is: it depends.

Applying the [principle of least privilege](https://en.wikipedia.org/wiki/Principle_of_least_privilege), all declarations other than those you plan to modify should use `const`.
The rationale is that if a variable didn't need to get written to, others working on the same codebase shouldn't automatically be able to write to the object, and will need to consider whether they really need to reassign to the variable.
Using `const` also makes code more predictable when reasoning about flow of data.

Use your best judgement, and if applicable, consult the matter with the rest of your team.

The majority of this handbook uses `let` declarations.

# Destructuring

Another ECMAScript 2015 feature that TypeScript has is destructuring.
For a complete reference, see [the article on the Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment).
In this section, we'll give a short overview.

## Array destructuring

The simplest form of destructuring is array destructuring assignment:

```ts
let input = [1, 2];
let [first, second] = input;
console.log(first); // outputs 1
console.log(second); // outputs 2
```

This creates two new variables named `first` and `second`.
This is equivalent to using indexing, but is much more convenient:

```ts
first = input[0];
second = input[1];
```

Destructuring works with already-declared variables as well:

```ts
// swap variables
[first, second] = [second, first];
```

And with parameters to a function:

```ts
function f([first, second]: [number, number]) {
    console.log(first);
    console.log(second);
}
f([1, 2]);
```

You can create a variable for the remaining items in a list using the syntax `...`:

```ts
let [first, ...rest] = [1, 2, 3, 4];
console.log(first); // outputs 1
console.log(rest); // outputs [ 2, 3, 4 ]
```

Of course, since this is JavaScript, you can just ignore trailing elements you don't care about:

```ts
let [first] = [1, 2, 3, 4];
console.log(first); // outputs 1
```

Or other elements:

```ts
let [, second, , fourth] = [1, 2, 3, 4];
console.log(second); // outputs 2
console.log(fourth); // outputs 4
```

## Tuple destructuring

Tuples may be destructured like arrays; the destructuring variables get the types of the corresponding tuple elements:

``` ts
let tuple: [number, string, boolean] = [7, "hello", true];

let [a, b, c] = tuple; // a: number, b: string, c: boolean
```

It's an error to destructure a tuple beyond the range of its elements:

``` ts
let [a, b, c, d] = tuple; // Error, no element at index 3
```

As with arrays, you can destructure the rest of the tuple with `...`, to get a shorter tuple:

``` ts
let [a, ...bc] = tuple; // bc: [string, boolean]
let [a, b, c, ...d] = tuple; // d: [], the empty tuple
```

Or ignore trailing elements, or other elements:

``` ts
let [a] = tuple; // a: number
let [, b] = tuple; // b: string
```

## Object destructuring

You can also destructure objects:

```ts
let o = {
    a: "foo",
    b: 12,
    c: "bar"
};
let { a, b } = o;
```

This creates new variables `a` and `b` from `o.a` and `o.b`.
Notice that you can skip `c` if you don't need it.

Like array destructuring, you can have assignment without declaration:

```ts
({ a, b } = { a: "baz", b: 101 });
```

Notice that we had to surround this statement with parentheses.
JavaScript normally parses a `{` as the start of block.

You can create a variable for the remaining items in an object using the syntax `...`:

```ts
let { a, ...passthrough } = o;
let total = passthrough.b + passthrough.c.length;

```

### Property renaming

You can also give different names to properties:

```ts
let { a: newName1, b: newName2 } = o;
```

Here the syntax starts to get confusing.
You can read `a: newName1` as "`a` as `newName1`".
The direction is left-to-right, as if you had written:

```ts
let newName1 = o.a;
let newName2 = o.b;
```

Confusingly, the colon here does *not* indicate the type.
The type, if you specify it, still needs to be written after the entire destructuring:

```ts
let { a, b }: { a: string, b: number } = o;
```

### Default values

Default values let you specify a default value in case a property is undefined:

```ts
function keepWholeObject(wholeObject: { a: string, b?: number }) {
    let { a, b = 1001 } = wholeObject;
}
```

In this example the `b?` indicates that `b` is optional, so it may be `undefined`.
`keepWholeObject` now has a variable for `wholeObject` as well as the properties `a` and `b`, even if `b` is undefined.

## Function declarations

Destructuring also works in function declarations.
For simple cases this is straightforward:

```ts
type C = { a: string, b?: number }
function f({ a, b }: C): void {
    // ...
}
```

But specifying defaults is more common for parameters, and getting defaults right with destructuring can be tricky.
First of all, you need to remember to put the pattern before the default value.

```ts
function f({ a="", b=0 } = {}): void {
    // ...
}
f();
```

> The snippet above is an example of type inference, explained later in the handbook.

Then, you need to remember to give a default for optional properties on the destructured property instead of the main initializer.
Remember that `C` was defined with `b` optional:

```ts
function f({ a, b = 0 } = { a: "" }): void {
    // ...
}
f({ a: "yes" }); // ok, default b = 0
f(); // ok, default to { a: "" }, which then defaults b = 0
f({}); // error, 'a' is required if you supply an argument
```

Use destructuring with care.
As the previous example demonstrates, anything but the simplest destructuring expression is confusing.
This is especially true with deeply nested destructuring, which gets *really* hard to understand even without piling on renaming, default values, and type annotations.
Try to keep destructuring expressions small and simple.
You can always write the assignments that destructuring would generate yourself.

## Spread

The spread operator is the opposite of destructuring.
It allows you to spread an array into another array, or an object into another object.
For example:

```ts
let first = [1, 2];
let second = [3, 4];
let bothPlus = [0, ...first, ...second, 5];
```

This gives bothPlus the value `[0, 1, 2, 3, 4, 5]`.
Spreading creates a shallow copy of `first` and `second`.
They are not changed by the spread.

You can also spread objects:

```ts
let defaults = { food: "spicy", price: "$$", ambiance: "noisy" };
let search = { ...defaults, food: "rich" };
```

Now `search` is `{ food: "rich", price: "$$", ambiance: "noisy" }`.
Object spreading is more complex than array spreading.
Like array spreading, it proceeds from left-to-right, but the result is still an object.
This means that properties that come later in the spread object overwrite properties that come earlier.
So if we modify the previous example to spread at the end:

```ts
let defaults = { food: "spicy", price: "$$", ambiance: "noisy" };
let search = { food: "rich", ...defaults };
```

Then the `food` property in `defaults` overwrites `food: "rich"`, which is not what we want in this case.

Object spread also has a couple of other surprising limits.
First, it only includes an objects'
[own, enumerable properties](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Enumerability_and_ownership_of_properties).
Basically, that means you lose methods when you spread instances of an object:

```ts
class C {
  p = 12;
  m() {
  }
}
let c = new C();
let clone = { ...c };
clone.p; // ok
clone.m(); // error!
```

Second, the Typescript compiler doesn't allow spreads of type parameters from generic functions.
That feature is expected in future versions of the language.
