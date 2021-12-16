# Documentação de testes da biblioteca [Redux.js](https://github.com/reduxjs/redux)

Redux é um gerenciador de estado previsível para aplicativos JavaScript.

Ele ajuda a escrever aplicativos que se comportam de maneira consistente, são executados em ambientes diferentes (cliente, servidor e nativo) e são fáceis de testar. Além disso, ele fornece uma ótima experiência para o desenvolvedor, como edição de código ao vivo combinada com um depurador de viagem no tempo.

É possível usar o Redux junto com o React ou com qualquer outra biblioteca de visualização. É minúsculo (2kB, incluindo dependências), mas tem um grande ecossistema de add-ons disponíveis.

**Serão documentados três testes:** dois testes da função createStore e os testes do utilitário compose

## createStore

Cria uma store do Redux que contém a árvore de estado completa do seu aplicativo. Deve haver apenas uma única loja em seu aplicativo.

Métodos auxiliares podem ser encontrados aqui: [reducers](https://github.com/reduxjs/redux/blob/master/test/helpers/reducers.ts) e [actionCreators: unknownAction e addTodo](https://github.com/reduxjs/redux/blob/master/test/helpers/actionCreators.ts)

``` ts
  it('applies the reducer to the initial state', () => {
    const store = createStore(reducers.todos, [
      {
        id: 1,
        text: 'Hello'
      }
    ])
    expect(store.getState()).toEqual([
      {
        id: 1,
        text: 'Hello'
      }
    ])

    store.dispatch(unknownAction())
    expect(store.getState()).toEqual([
      {
        id: 1,
        text: 'Hello'
      }
    ])

    store.dispatch(addTodo('World'))
    expect(store.getState()).toEqual([
      {
        id: 1,
        text: 'Hello'
      },
      {
        id: 2,
        text: 'World'
      }
    ])
  })
```
Esse teste verifica a capacidade de uma store de aplicar um reducer a um estado inicial. Para isso, ele utiliza o reducer de todos na inicialização da store e dispara a action unknownAction. É esperado que para uma action desconhecida o estado resultante seja o mesmo que inicial. Logo após é disparada a action addTodo('World'). Esta action deve ser capturada pelo reducer de todos que foi aplicado pela store e modificar o estado adicionando o todo "World" para a sua lista.

``` ts
  it('preserves the state when replacing a reducer', () => {
    const store = createStore(reducers.todos)
    store.dispatch(addTodo('Hello'))
    store.dispatch(addTodo('World'))
    expect(store.getState()).toEqual([
      {
        id: 1,
        text: 'Hello'
      },
      {
        id: 2,
        text: 'World'
      }
    ])

    store.replaceReducer(reducers.todosReverse)
    expect(store.getState()).toEqual([
      {
        id: 1,
        text: 'Hello'
      },
      {
        id: 2,
        text: 'World'
      }
    ])

    store.dispatch(addTodo('Perhaps'))
    expect(store.getState()).toEqual([
      {
        id: 3,
        text: 'Perhaps'
      },
      {
        id: 1,
        text: 'Hello'
      },
      {
        id: 2,
        text: 'World'
      }
    ])

    store.replaceReducer(reducers.todos)
    expect(store.getState()).toEqual([
      {
        id: 3,
        text: 'Perhaps'
      },
      {
        id: 1,
        text: 'Hello'
      },
      {
        id: 2,
        text: 'World'
      }
    ])

    store.dispatch(addTodo('Surely'))
    expect(store.getState()).toEqual([
      {
        id: 3,
        text: 'Perhaps'
      },
      {
        id: 1,
        text: 'Hello'
      },
      {
        id: 2,
        text: 'World'
      },
      {
        id: 4,
        text: 'Surely'
      }
    ])
  })
```
Esse teste confere se ao substituir um reducer por outro a store garante que o estado atual não será afetado. Para isso são disparadas actions com o reducer todos que adiciona os items no final da lista. Em seguida o reducer é substituído pelo todosReverse que adiciona os items no início da lista e é disparada uma action. Ao final o reducer é substituído de volta para o todos e é verificado após o disparo de uma action se o estado é o esperado.

## compose

Compõe funções da direita para a esquerda.

Este é um utilitário de programação funcional e está incluído no Redux por conveniência. Pode ser utilizado para aplicar vários otimizadores de store em uma fileira.

``` ts

describe('Utils', () => {
  describe('compose', () => {
    it('composes from right to left', () => {
      const double = (x: number) => x * 2
      const square = (x: number) => x * x
      expect(compose(square)(5)).toBe(25)
      expect(compose(square, double)(5)).toBe(100)
      expect(compose(double, square, double)(5)).toBe(200)
    })

    it('composes functions from right to left', () => {
      const a = (next: (x: string) => string) => (x: string) => next(x + 'a')
      const b = (next: (x: string) => string) => (x: string) => next(x + 'b')
      const c = (next: (x: string) => string) => (x: string) => next(x + 'c')
      const final = (x: string) => x

      expect(compose(a, b, c)(final)('')).toBe('abc')
      expect(compose(b, c, a)(final)('')).toBe('bca')
      expect(compose(c, a, b)(final)('')).toBe('cab')
    })

    it('throws at runtime if argument is not a function', () => {
      type sFunc = (x: number, y: number) => number
      const square = (x: number, _: number) => x * x
      const add = (x: number, y: number) => x + y

      expect(() =>
        compose(square, add, false as unknown as sFunc)(1, 2)
      ).toThrow()
      // @ts-expect-error
      expect(() => compose(square, add, undefined)(1, 2)).toThrow()
      expect(() =>
        compose(square, add, true as unknown as sFunc)(1, 2)
      ).toThrow()
      expect(() =>
        compose(square, add, NaN as unknown as sFunc)(1, 2)
      ).toThrow()
      expect(() =>
        compose(square, add, '42' as unknown as sFunc)(1, 2)
      ).toThrow()
    })

    it('can be seeded with multiple arguments', () => {
      const square = (x: number, _: number) => x * x
      const add = (x: number, y: number) => x + y
      expect(compose(square, add)(1, 2)).toBe(9)
    })

    it('returns the first given argument if given no functions', () => {
      expect(compose<number>()(1, 2)).toBe(1)
      expect(compose()(3)).toBe(3)
      expect(compose()(undefined)).toBe(undefined)
    })

    it('returns the first function if given only one', () => {
      const fn = () => {}

      expect(compose(fn)).toBe(fn)
    })
  })
})
```

Os testes da função compose são bem auto explicativos. O primeiro verifica a composição de funções da direita para esquerda de uma forma mais flexível. O segundo utiliza strings para conferir a ordem em que as funções são executadas. O terceiro testa se a função gera uma exceção caso um dos argumentos não seja uma função. O quarto verifica se o compose repassa múltiplos argumentos para a primeira função de sua composição. O quinto checa se caso nenhum argumento é passado para o compose, o seu resultado deverá ser a função identidade, ou seja, retornar o argumento que lhe foi passado. Por último, caso  seja passada somente uma função para o compose ele deverá retornar exatamente a mesma função como resultado.
