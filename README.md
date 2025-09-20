Esse script implementa um slider simples: seleciona todas as imagens (ou slides), controla qual slide está visível com um índice (imagemAtual) e alterna entre slides ao clicar nas setas de avançar/voltar. A classe CSS 'mostrar' é usada para exibir o slide atual; a classe 'opacidade' controla a aparência (provavelmente desabilitada/“apagada”) das setas.

## 1) Seletores e variáveis iniciais
~~~
const imagens = document.querySelectorAll('.slide');
const setaAvancar = document.getElementById('seta-avancar');
const setaVoltar = document.getElementById('seta-voltar');

let imagemAtual = 0;
~~~

imagens: pega todos os elementos do DOM que têm a classe .slide. O retorno é um NodeList (array-like), então imagens.length e imagens[0] funcionam. É uma lista estática (em querySelectorAll) — não atualiza automaticamente se você adicionar/remover slides depois.

setaAvancar / setaVoltar: referenciam os elementos das setas pelos id. Se não existir um elemento com esse id, a variável será null.

imagemAtual = 0: índice do slide atual (zero-based). 0 significa que começamos pelo primeiro slide.

Observação prática: é bom checar se os seletores realmente encontraram elementos antes de usar (if (!setaAvancar) { /* tratar */ }), para evitar erros no console.

---

## 2) Evento de avançar
~~~
setaAvancar.addEventListener('click', function() {
    if(imagemAtual === imagens.length -1) {
        return;
    }   
    imagemAtual++;

   esconderImagemAberta();
   mostrarImagem();
   mostrarOuEsconderSetas();
});
~~~

### Passo a passo:
~~~
setaAvancar.addEventListener('click', ...) — adiciona um listener para o evento de clique na seta de avançar.

if(imagemAtual === imagens.length -1) { return; } — verifica se já estamos no último slide (o último índice é length - 1). Se sim, interrompe a função (o return evita avançar além do último).

imagemAtual++; — incrementa o índice para o próximo slide.

esconderImagemAberta(); — remove a classe .mostrar do slide atualmente visível (veja função abaixo).

mostrarImagem(); — adiciona .mostrar ao novo slide (imagens[imagemAtual]), tornando-o visível.

mostrarOuEsconderSetas(); — atualiza as classes das setas (ex.: habilita/desabilita visualmente as setas dependendo do índice).
~~~

**Nota sobre ordem: aqui o código incrementa imagemAtual antes de chamar esconderImagemAberta(). Isso funciona porque esconderImagemAberta() busca o elemento com .mostrar no DOM (independente do índice). A ordem é um pouco inconsistente com o evento de voltar (ver abaixo), mas funcional.**

---

## 3) Função mostrarImagem
~~~
function mostrarImagem() {
    imagens[imagemAtual].classList.add('mostrar');
}
~~~

Acessa o elemento no array imagens pelo índice imagemAtual e adiciona a classe CSS 'mostrar'.

Presume que imagens[imagemAtual] existe (ou seja, imagemAtual está dentro do intervalo). Se imagens estiver vazio ou imagemAtual fora do intervalo, dará erro.

---

## 4) Função esconderImagemAberta
~~~
function esconderImagemAberta() {
    const imagemAberta = document.querySelector('.mostrar');
    imagemAberta.classList.remove('mostrar');
}
~~~

Procura no DOM o elemento que atualmente tem a classe .mostrar (o slide visível).

Remove a classe .mostrar desse elemento, escondendo-o (conforme o CSS).

**Risco: se não houver nenhum elemento com .mostrar (por exemplo, inicialização incorreta), imagemAberta será null e imagemAberta.classList.remove causará um erro (Cannot read property 'classList' of null).
Melhoria recomendada: checar if (imagemAberta) imagemAberta.classList.remove('mostrar');**

---

## 5) Função mostrarOuEsconderSetas
~~~
function mostrarOuEsconderSetas() {
    const naoEhAPrimeiraImagem = imagemAtual !== 0;
    if(naoEhAPrimeiraImagem) {
        setaVoltar.classList.remove('opacidade');
    } else {
        setaVoltar.classList.add('opacidade');
    }

    const chegouNaUltimaImagem = imagemAtual !== 0 && imagemAtual === imagens.length -1;
    if(chegouNaUltimaImagem) {
        setaAvancar.classList.add('opacidade');
    } else {
        setaAvancar.classList.remove('opacidade');
    }
}
~~~

naoEhAPrimeiraImagem: verifica se não estamos no primeiro slide.

Se for verdade, remove a classe 'opacidade' da seta de voltar (ou seja, torna a seta visível/ativa).

Caso contrário (estamos no primeiro), adiciona 'opacidade' para "desabilitar" visualmente a seta de voltar.

chegouNaUltimaImagem: expressão que será true se imagemAtual for o último índice e imagemAtual !== 0.

Se for true, adiciona 'opacidade' à seta de avançar (esconde/desabilita a seta).

Caso contrário, remove 'opacidade' da seta de avançar.

### Observações:

A condição imagemAtual !== 0 && imagemAtual === imagens.length -1 impede que a seta de avançar seja escondida quando há apenas um slide (pois se há somente um slide, imagemAtual === 0 e também imagemAtual === imagens.length - 1 — mas a conjunção imagemAtual !== 0 torna o resultado false). Talvez isso seja intencional (manter ambas as setas opacas quando só existe um slide) — apenas fique ciente do comportamento.

opacidade é uma classe visual — o script espera que o CSS trate dela (por exemplo opacity: 0.3; pointer-events: none;).

---

## 6) Evento de voltar
~~~
setaVoltar.addEventListener('click', function() {   
    if (imagemAtual === 0) {
       return;
   }
    esconderImagemAberta();

    imagemAtual--;

    mostrarImagem();
   mostrarOuEsconderSetas();
});
~~~

### Passo a passo:

Se imagemAtual === 0 (primeiro slide), return — impede voltar antes do primeiro.

esconderImagemAberta(); — remove .mostrar do slide atual.

imagemAtual--; — decrementa o índice para o slide anterior.

mostrarImagem(); — adiciona .mostrar ao novo slide (imagens[imagemAtual]).

mostrarOuEsconderSetas(); — atualiza visuais das setas.

Observação sobre ordem: aqui o código chama esconderImagemAberta() antes de fazer imagemAtual--. No bloco de avançar o imagemAtual foi incrementado antes da chamada de esconderImagemAberta(). Ambos funcionam porque esconderImagemAberta() busca .mostrar diretamente no DOM, mas a ordem fica inconsistente. Eu recomendo padronizar: primeiro remover o .mostrar, depois atualizar o índice, depois adicionar .mostrar ao novo índice — isso facilita raciocínio e possíveis refatorações.

---

## 7) Fluxo de execução (resumido)

Ao clicar em avançar:

verifica limite; incrementa imagemAtual; remove classe .mostrar do slide atual; adiciona .mostrar ao slide novo; atualiza setas.

Ao clicar em voltar:

verifica limite; remove .mostrar do slide atual; decrementa imagemAtual; adiciona .mostrar ao slide novo; atualiza setas.
