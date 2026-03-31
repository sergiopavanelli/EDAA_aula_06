# Resoluções — Aula 06: TADs Lineares (Vec, Pilha, Fila, Deque)

## Aluno: Sérgio Pinton Pavanelli
## RA: 123220202

**Unidade Curricular:** Estruturas de Dados e Análise de Algoritmos  
**Código:** 0006963  
**Data:** 30/03/2026  


---

## Grupo 1 — Vec e Operações Básicas

### 1. Inversão com Vec

**Complexidade:** O(n) tempo · O(n) espaço extra (Vec auxiliar)

```rust
fn main() {
    let mut original = vec![1, 2, 3, 4, 5];
    let mut invertido: Vec<i32> = Vec::new();

    // Pop de `original` empilha em `invertido` na ordem inversa
    while let Some(val) = original.pop() {   // O(1) amortizado por pop
        invertido.push(val);                  // O(1) amortizado por push
    }

    println!("{:?}", invertido); // [5, 4, 3, 2, 1]
}
```

> `pop()` remove o último elemento em O(1) amortizado. Fazendo n pops e n pushes: **O(n)** no total.

---

### 2. Contador de ocorrências

**Complexidade:** O(n) tempo · O(k) espaço — k = número de letras distintas

```rust
use std::collections::HashMap;

fn main() {
    let frase = "estrutura de dados";
    let vec: Vec<char> = frase.chars().collect();

    let mut contagem: HashMap<char, u32> = HashMap::new();

    for x in &vec {                          // O(n) iterações
        if *x != ' ' {
            *contagem.entry(*x).or_insert(0) += 1; // O(1) amortizado (hash)
        }
    }

    let mut pares: Vec<(char, u32)> = contagem.into_iter().collect();
    pares.sort_by_key(|&(c, _)| c);         // ordenar para saída legível
    for (letra, qtd) in pares {
        println!("'{}': {}", letra, qtd);
    }
}
```

---

### 3. Remoção condicional (sem `.retain()`)

**Complexidade:** O(n) tempo · O(n) espaço — cria novo Vec

```rust
fn remover_pares(v: Vec<i32>) -> Vec<i32> {
    // Coleta apenas os ímpares em um novo Vec — O(n)
    let mut resultado: Vec<i32> = Vec::new();
    for &x in &v {
        if x % 2 != 0 {
            resultado.push(x); // O(1) amortizado
        }
    }
    resultado
}

fn main() {
    let numeros = vec![1, 2, 3, 4, 5, 6, 7, 8];
    println!("{:?}", remover_pares(numeros)); // [1, 3, 5, 7]
}
```

> **Análise de alternativas:**
> - Remoção in-place com `remove(i)` seria **O(n²)** — cada `remove` desloca todos os elementos à direita.
> - Criar novo Vec (abordagem acima) é **O(n)** mas usa espaço extra.
> - `.retain()` faz o mesmo que a abordagem acima internamente (O(n)), apenas não está disponível aqui por restrição do enunciado.

---

### 4. Mescla ordenada

**Complexidade:** O((n+m) log(n+m)) com `extend+sort` · O(n+m) com mescla manual

```rust
// Versão 1: extend + sort — O((n+m) log(n+m))
fn mescla_sort(a: &[i32], b: &[i32]) -> Vec<i32> {
    let mut resultado = Vec::with_capacity(a.len() + b.len());
    resultado.extend_from_slice(a);    // O(n)
    resultado.extend_from_slice(b);    // O(m)
    resultado.sort();                  // O((n+m) log(n+m))
    resultado
}

// Versão 2: mescla manual — O(n+m) aproveitando a ordem dos vetores
fn mescla_manual(a: &[i32], b: &[i32]) -> Vec<i32> {
    let mut resultado = Vec::with_capacity(a.len() + b.len());
    let (mut i, mut j) = (0, 0);
    while i < a.len() && j < b.len() {   // O(n+m)
        if a[i] <= b[j] {
            resultado.push(a[i]);
            i += 1;
        } else {
            resultado.push(b[j]);
            j += 1;
        }
    }
    resultado.extend_from_slice(&a[i..]); // restante de a
    resultado.extend_from_slice(&b[j..]); // restante de b
    resultado
}

fn main() {
    let a = vec![1, 3, 5, 7];
    let b = vec![2, 4, 6, 8];
    println!("extend+sort:  {:?}", mescla_sort(&a, &b));
    println!("mescla manual:{:?}", mescla_manual(&a, &b));
}
```

---

## Grupo 2 — Pilha (Stack)

### 5. Calculadora RPN

**Complexidade:** O(n) tempo · O(n) espaço — n = número de tokens

```rust
fn avaliar_rpn(expressao: &str) -> f64 {
    let mut pilha: Vec<f64> = Vec::new();

    for token in expressao.split_whitespace() { // O(n) tokens
        match token {
            "+" | "-" | "*" | "/" => {
                let b = pilha.pop().expect("pilha vazia");  // O(1)
                let a = pilha.pop().expect("pilha vazia");  // O(1)
                let resultado = match token {
                    "+" => a + b,
                    "-" => a - b,
                    "*" => a * b,
                    "/" => a / b,
                    _   => unreachable!(),
                };
                pilha.push(resultado); // O(1)
            }
            numero => {
                pilha.push(numero.parse::<f64>().expect("token inválido")); // O(1)
            }
        }
    }

    pilha.pop().expect("expressão inválida")
}

fn main() {
    // "3 4 + 2 *" → (3 + 4) * 2 = 14
    println!("{}", avaliar_rpn("3 4 + 2 *")); // 14
    // "5 1 2 + 4 * + 3 -" → 14
    println!("{}", avaliar_rpn("5 1 2 + 4 * + 3 -")); // 14
}
```

---

### 6. Histórico de navegação

**Complexidade:** O(1) por operação (push/pop em Vec)

```rust
struct Navegador {
    historico_back: Vec<String>,
    historico_forward: Vec<String>,
    atual: Option<String>,
}

impl Navegador {
    fn new() -> Self {
        Navegador { historico_back: vec![], historico_forward: vec![], atual: None }
    }

    // Visitar nova página — limpa o histórico de avanço
    fn visitar(&mut self, url: String) {
        if let Some(pagina) = self.atual.take() {
            self.historico_back.push(pagina); // O(1) amortizado
        }
        self.historico_forward.clear();
        self.atual = Some(url);
    }

    // Voltar — move atual para forward, topo de back vira atual
    fn voltar(&mut self) {
        if let Some(anterior) = self.historico_back.pop() { // O(1)
            if let Some(pagina) = self.atual.take() {
                self.historico_forward.push(pagina);         // O(1)
            }
            self.atual = Some(anterior);
        }
    }

    // Avançar — move atual para back, topo de forward vira atual
    fn avancar(&mut self) {
        if let Some(proxima) = self.historico_forward.pop() { // O(1)
            if let Some(pagina) = self.atual.take() {
                self.historico_back.push(pagina);             // O(1)
            }
            self.atual = Some(proxima);
        }
    }

    fn pagina_atual(&self) -> &str {
        self.atual.as_deref().unwrap_or("(sem página)")
    }
}

fn main() {
    let mut nav = Navegador::new();
    nav.visitar("google.com".into());
    nav.visitar("rust-lang.org".into());
    nav.visitar("doc.rust-lang.org".into());
    println!("{}", nav.pagina_atual()); // doc.rust-lang.org
    nav.voltar();
    println!("{}", nav.pagina_atual()); // rust-lang.org
    nav.avancar();
    println!("{}", nav.pagina_atual()); // doc.rust-lang.org
}
```

---

### 7. Desfazer/Refazer

**Complexidade:** O(1) amortizado para todas as operações

```rust
struct Editor {
    texto_atual: String,
    pilha_undo: Vec<String>,
    pilha_redo: Vec<String>,
}

impl Editor {
    fn new() -> Self {
        Editor { texto_atual: String::new(), pilha_undo: vec![], pilha_redo: vec![] }
    }

    fn digitar(&mut self, texto: &str) {
        self.pilha_undo.push(self.texto_atual.clone()); // salva estado atual
        self.pilha_redo.clear();                         // nova ação invalida redo
        self.texto_atual.push_str(texto);
    }

    fn desfazer(&mut self) {
        if let Some(anterior) = self.pilha_undo.pop() {      // O(1)
            self.pilha_redo.push(self.texto_atual.clone());   // O(1)
            self.texto_atual = anterior;
        }
    }

    fn refazer(&mut self) {
        if let Some(proximo) = self.pilha_redo.pop() {       // O(1)
            self.pilha_undo.push(self.texto_atual.clone());   // O(1)
            self.texto_atual = proximo;
        }
    }
}

fn main() {
    let mut ed = Editor::new();
    ed.digitar("Olá");
    ed.digitar(", mundo");
    println!("{}", ed.texto_atual); // "Olá, mundo"
    ed.desfazer();
    println!("{}", ed.texto_atual); // "Olá"
    ed.refazer();
    println!("{}", ed.texto_atual); // "Olá, mundo"
}
```

---

### 8. Sequências de símbolos balanceadas

**Complexidade:** O(n) tempo · O(n) espaço

```rust
fn balanceado(expressao: &str) -> bool {
    let mut pilha: Vec<char> = Vec::new();

    for c in expressao.chars() {               // O(n)
        match c {
            '(' | '[' | '{' => pilha.push(c),  // O(1)
            ')' => { if pilha.pop() != Some('(') { return false; } }
            ']' => { if pilha.pop() != Some('[') { return false; } }
            '}' => { if pilha.pop() != Some('{') { return false; } }
            _   => {}
        }
    }

    pilha.is_empty() // pilha vazia = todos os delimitadores fechados
}

fn main() {
    let casos = ["{[()]}", "([)]", "((("];
    for expr in casos {
        println!("{:?} → {}", expr, if balanceado(expr) { "balanceado" } else { "inválido" });
    }
    // "{[()]}" → balanceado
    // "([)]"   → inválido
    // "((("    → inválido
}
```

---

### 9. Pilha com mínimo (StackMin)

**Complexidade:** O(1) para push, pop e min

```rust
struct StackMin {
    data: Vec<i32>,
    mins: Vec<i32>, // topo sempre contém o mínimo atual
}

impl StackMin {
    fn new() -> Self {
        StackMin { data: vec![], mins: vec![] }
    }

    fn push(&mut self, val: i32) {
        self.data.push(val);                                       // O(1)
        let novo_min = match self.mins.last() {
            Some(&m) => m.min(val),
            None     => val,
        };
        self.mins.push(novo_min);                                  // O(1)
    }

    fn pop(&mut self) -> Option<i32> {
        self.mins.pop();                                           // O(1)
        self.data.pop()                                            // O(1)
    }

    fn min(&self) -> Option<i32> {
        self.mins.last().copied()                                  // O(1)
    }
}

fn main() {
    let mut s = StackMin::new();
    s.push(5);
    s.push(3);
    s.push(7);
    s.push(2);
    println!("min = {}", s.min().unwrap()); // 2
    s.pop();
    println!("min = {}", s.min().unwrap()); // 3
    s.pop();
    println!("min = {}", s.min().unwrap()); // 3
}
```

---

## Grupo 3 — Fila (Queue)

### 10. Simulador de fila de banco

**Complexidade:** O(n) tempo · O(n) espaço — n = número de clientes

```rust
use std::collections::VecDeque;

fn main() {
    // Tempos de chegada e duração de atendimento (em minutos)
    let chegadas: Vec<u32> = vec![0, 2, 5, 7, 10, 12];
    let duracoes: Vec<u32> = vec![4, 3, 5, 2, 3, 4];

    let mut fila: VecDeque<(u32, u32)> = VecDeque::new(); // (chegada, duracao)
    for i in 0..chegadas.len() {
        fila.push_back((chegadas[i], duracoes[i]));        // O(1)
    }

    let mut tempo_atual = 0u32;
    let mut total_espera = 0u32;
    let n = fila.len();

    while let Some((chegada, duracao)) = fila.pop_front() { // O(1) por dequeue
        if tempo_atual < chegada {
            tempo_atual = chegada; // atendente ficou ocioso
        }
        let espera = tempo_atual - chegada;
        total_espera += espera;
        println!("Cliente chegou em t={}: esperou {} min, atendido em t={}",
            chegada, espera, tempo_atual);
        tempo_atual += duracao;
    }

    println!("Tempo médio de espera: {:.2} min", total_espera as f64 / n as f64);
}
```

---

### 11. Impressora compartilhada

**Complexidade:** O(n) — cada job é enfileirado e removido uma vez

```rust
use std::collections::VecDeque;

struct Job {
    nome: String,
    paginas: u32,
}

fn main() {
    let mut fila: VecDeque<Job> = VecDeque::new();

    // Chegada de trabalhos
    fila.push_back(Job { nome: "Relatório".into(), paginas: 10 }); // O(1)
    fila.push_back(Job { nome: "Currículo".into(), paginas: 2  });
    fila.push_back(Job { nome: "TCC".into(),       paginas: 80 });

    // Processamento em ordem de chegada
    while let Some(job) = fila.pop_front() { // O(1)
        println!("Imprimindo '{}' — {} página(s)", job.nome, job.paginas);
    }
}
```

---

### 12. Buffer de mensagens (FilaCircular com overwrite)

**Complexidade:** O(1) para enqueue e dequeue

```rust
use std::collections::VecDeque;

struct FilaCircular {
    buffer: VecDeque<String>,
    capacidade: usize,
}

impl FilaCircular {
    fn new(capacidade: usize) -> Self {
        FilaCircular { buffer: VecDeque::with_capacity(capacidade), capacidade }
    }

    // Se cheio, descarta a mensagem mais antiga (overwrite)
    fn enqueue(&mut self, msg: String) {
        if self.buffer.len() == self.capacidade {
            let descartada = self.buffer.pop_front(); // O(1) — descarta a mais antiga
            println!("[overwrite] descartando: {:?}", descartada.unwrap());
        }
        self.buffer.push_back(msg); // O(1)
    }

    fn dequeue(&mut self) -> Option<String> {
        self.buffer.pop_front() // O(1)
    }
}

fn main() {
    let mut buf = FilaCircular::new(3);
    buf.enqueue("msg1".into());
    buf.enqueue("msg2".into());
    buf.enqueue("msg3".into());
    buf.enqueue("msg4".into()); // overwrite: descarta msg1
    while let Some(m) = buf.dequeue() {
        println!("lendo: {}", m); // msg2, msg3, msg4
    }
}
```

---

### 13. Fila de prioridade manual

**Complexidade:** enqueue O(1) · dequeue O(n) — busca linear pelo maior

```rust
use std::collections::VecDeque;

struct Item {
    valor: String,
    prioridade: u32,
}

struct FilaPrioridade {
    dados: VecDeque<Item>,
}

impl FilaPrioridade {
    fn new() -> Self { FilaPrioridade { dados: VecDeque::new() } }

    fn enqueue(&mut self, valor: String, prioridade: u32) {
        self.dados.push_back(Item { valor, prioridade }); // O(1)
    }

    // Remove o item de maior prioridade; entre iguais, o mais antigo (FIFO)
    fn dequeue(&mut self) -> Option<String> {
        if self.dados.is_empty() { return None; }
        // Busca linear pelo índice do maior priority — O(n)
        let idx_max = self.dados.iter()
            .enumerate()
            .max_by_key(|(_, item)| item.prioridade)
            .map(|(i, _)| i)
            .unwrap();
        self.dados.remove(idx_max).map(|item| item.valor) // O(n) pelo shift
    }
}

fn main() {
    let mut fila = FilaPrioridade::new();
    fila.enqueue("normal A".into(), 1);
    fila.enqueue("urgente".into(),  3);
    fila.enqueue("normal B".into(), 1);
    fila.enqueue("importante".into(), 2);

    while let Some(val) = fila.dequeue() {
        println!("{}", val);
    }
    // urgente → importante → normal A → normal B
}
```

---

## Grupo 4 — Deque (Double-Ended Queue)

### 14. Palíndromo com Deque

**Complexidade:** O(n) tempo · O(n) espaço

```rust
use std::collections::VecDeque;

fn e_palindromo(s: &str) -> bool {
    // Filtra espaços e converte para minúsculo — O(n)
    let mut deque: VecDeque<char> = s
        .chars()
        .filter(|c| !c.is_whitespace())
        .map(|c| c.to_lowercase().next().unwrap())
        .collect();

    // Compara extremos até sobrar ≤ 1 elemento — O(n/2)
    while deque.len() > 1 {
        if deque.pop_front() != deque.pop_back() { // O(1) cada
            return false;
        }
    }
    true
}

fn main() {
    let casos = ["A man a plan a canal Panama", "racecar", "hello"];
    for s in casos {
        println!("{:?} → {}", s, if e_palindromo(s) { "palíndromo" } else { "não é" });
    }
}
```

---

### 15. Janela deslizante máxima

**Complexidade:** O(n) tempo · O(k) espaço — k = tamanho da janela

```rust
use std::collections::VecDeque;

fn janela_maxima(v: &[i32], k: usize) -> Vec<i32> {
    let mut deque: VecDeque<usize> = VecDeque::new(); // índices (invariante: decrescente por valor)
    let mut resultado = Vec::new();

    for i in 0..v.len() {
        // Remove índices fora da janela
        if let Some(&front) = deque.front() {
            if front + k <= i {
                deque.pop_front(); // O(1)
            }
        }
        // Remove do fundo índices menores que o atual (não podem ser máximo)
        while let Some(&back) = deque.back() {
            if v[back] <= v[i] {
                deque.pop_back(); // O(1) — cada elemento é removido no máximo 1x
            } else {
                break;
            }
        }
        deque.push_back(i); // O(1)

        // Janela completa: registra o máximo (topo do deque)
        if i + 1 >= k {
            resultado.push(v[*deque.front().unwrap()]);
        }
    }
    resultado
}

fn main() {
    let v = vec![1, 3, -1, -3, 5, 3, 6, 7];
    println!("{:?}", janela_maxima(&v, 3)); // [3, 3, 5, 5, 6, 7]
}
```

---

### 16. Fila de tarefas com prioridade de frente

**Complexidade:** O(1) para todas as operações

```rust
use std::collections::VecDeque;

struct FilaTarefas {
    deque: VecDeque<String>,
}

impl FilaTarefas {
    fn new() -> Self { FilaTarefas { deque: VecDeque::new() } }

    fn adicionar_urgente(&mut self, tarefa: String) {
        self.deque.push_front(tarefa); // O(1)
    }

    fn adicionar_normal(&mut self, tarefa: String) {
        self.deque.push_back(tarefa); // O(1)
    }

    fn processar(&mut self) -> Option<String> {
        self.deque.pop_front() // O(1) — sempre da frente
    }
}

fn main() {
    let mut ft = FilaTarefas::new();
    ft.adicionar_normal("build projeto".into());
    ft.adicionar_normal("rodar testes".into());
    ft.adicionar_urgente("corrigir bug crítico".into());
    ft.adicionar_urgente("deploy hotfix".into());

    while let Some(t) = ft.processar() {
        println!("Processando: {}", t);
    }
    // deploy hotfix → corrigir bug crítico → build projeto → rodar testes
}
```

---

## Grupo 5 — Reflexão e Análise

### 17. Comparação de desempenho

**Complexidade:** Vec ingênua O(n²) total · VecDeque O(n) total · FilaCircular O(n) total

```rust
use std::collections::VecDeque;
use std::time::Instant;

const N: usize = 10_000;

fn benchmark_vec() -> (u128, u128) {
    let mut v: Vec<i32> = Vec::new();
    let t0 = Instant::now();
    for i in 0..N as i32 { v.push(i); }               // O(1) amortizado cada
    let t_enqueue = t0.elapsed().as_micros();

    let t1 = Instant::now();
    while !v.is_empty() { v.remove(0); }               // O(n) cada → O(n²) total
    let t_dequeue = t1.elapsed().as_micros();
    (t_enqueue, t_dequeue)
}

fn benchmark_vecdeque() -> (u128, u128) {
    let mut d: VecDeque<i32> = VecDeque::new();
    let t0 = Instant::now();
    for i in 0..N as i32 { d.push_back(i); }           // O(1)
    let t_enqueue = t0.elapsed().as_micros();

    let t1 = Instant::now();
    while d.pop_front().is_some() {}                    // O(1) cada → O(n) total
    let t_dequeue = t1.elapsed().as_micros();
    (t_enqueue, t_dequeue)
}

// Fila circular manual com array de tamanho fixo
struct FilaCircularArray {
    buf: Vec<i32>,
    head: usize,
    tail: usize,
    tamanho: usize,
    cap: usize,
}

impl FilaCircularArray {
    fn new(cap: usize) -> Self {
        FilaCircularArray { buf: vec![0; cap], head: 0, tail: 0, tamanho: 0, cap }
    }
    fn enqueue(&mut self, val: i32) -> bool {
        if self.tamanho == self.cap { return false; }
        self.buf[self.tail] = val;
        self.tail = (self.tail + 1) % self.cap;         // O(1)
        self.tamanho += 1;
        true
    }
    fn dequeue(&mut self) -> Option<i32> {
        if self.tamanho == 0 { return None; }
        let val = self.buf[self.head];
        self.head = (self.head + 1) % self.cap;         // O(1)
        self.tamanho -= 1;
        Some(val)
    }
}

fn benchmark_circular() -> (u128, u128) {
    let mut fc = FilaCircularArray::new(N);
    let t0 = Instant::now();
    for i in 0..N as i32 { fc.enqueue(i); }
    let t_enqueue = t0.elapsed().as_micros();

    let t1 = Instant::now();
    while fc.dequeue().is_some() {}
    let t_dequeue = t1.elapsed().as_micros();
    (t_enqueue, t_dequeue)
}

fn main() {
    let (eq, dq) = benchmark_vec();
    println!("Vec ingênua    — enqueue: {}µs  dequeue: {}µs", eq, dq);
    let (eq, dq) = benchmark_vecdeque();
    println!("VecDeque       — enqueue: {}µs  dequeue: {}µs", eq, dq);
    let (eq, dq) = benchmark_circular();
    println!("FilaCircular   — enqueue: {}µs  dequeue: {}µs", eq, dq);
}
```

> **Resultado esperado:** `Vec` com `remove(0)` é drasticamente mais lento no dequeue — O(n²) total vs O(n) das outras duas.

---

### 18. Quando usar qual TAD?

| Cenário | TAD recomendado | Justificativa |
|---------|----------------|---------------|
| **(a)** Botão "Ctrl+Z" de editor | **Pilha** | Desfazer segue LIFO: a última ação é desfeita primeiro. Uma segunda pilha gerencia Ctrl+Y. |
| **(b)** Processar pedidos de restaurante em ordem | **Fila** | Pedidos chegam e saem em ordem FIFO: o primeiro a chegar é o primeiro a ser preparado. |
| **(c)** Verificar tags bem formadas em HTML | **Pilha** | Estrutura de aninhamento de tags é LIFO: a tag aberta mais recente deve ser a próxima a fechar. |
| **(d)** Navegar em diretório em largura (BFS) | **Fila** | BFS processa nós nível a nível, usando FIFO: enfileira filhos antes de processar o próximo nível. |
| **(e)** Verificar se sequência de palavras é palíndromo | **Deque** | Comparação de extremos: retira palavra do início e do fim simultaneamente em O(1) por operação. |

---

### 19. Fila com iteração controlada

**Complexidade:** O(n) tempo — cada elemento é processado uma vez

```rust
use std::collections::VecDeque;

// Processa `tamanho_lote` elementos por vez, imprimindo cada lote
fn processar_em_lotes(fila: &mut VecDeque<i32>, tamanho_lote: usize) {
    let mut lote_num = 1;
    while !fila.is_empty() {
        let mut lote = Vec::new();
        for _ in 0..tamanho_lote {               // O(tamanho_lote) por lote
            match fila.pop_front() {              // O(1)
                Some(val) => lote.push(val),
                None => break,
            }
        }
        println!("Lote {}: {:?}", lote_num, lote);
        lote_num += 1;
    }
}

fn main() {
    let mut fila: VecDeque<i32> = (1..=10).collect();
    processar_em_lotes(&mut fila, 3);
    // Lote 1: [1, 2, 3]
    // Lote 2: [4, 5, 6]
    // Lote 3: [7, 8, 9]
    // Lote 4: [10]
}
```

---

### 20. Mini-projeto — Round Robin

**Complexidade:** O(n · ⌈t_max/Q⌉) tempo · O(n) espaço — n = processos, t_max = maior burst time

```rust
use std::collections::VecDeque;

struct Processo {
    nome: String,
    tempo_restante: u32,
    tempo_chegada: u32,
}

fn round_robin(processos: Vec<Processo>, quantum: u32) {
    let mut fila: VecDeque<Processo> = processos.into();
    let mut tempo_atual = 0u32;

    println!("=== Simulação Round Robin (quantum = {}) ===", quantum);

    while let Some(mut proc) = fila.pop_front() { // O(1)
        // Processa por no máximo `quantum` unidades de tempo
        let executado = proc.tempo_restante.min(quantum);
        proc.tempo_restante -= executado;
        tempo_atual += executado;

        if proc.tempo_restante == 0 {
            println!(
                "Processo '{}' concluído no tempo t={} (chegou em t={})",
                proc.nome, tempo_atual, proc.tempo_chegada
            );
        } else {
            fila.push_back(proc); // ainda tem trabalho: volta para o fim da fila
        }
    }
}

fn main() {
    let processos = vec![
        Processo { nome: "P1".into(), tempo_restante: 8,  tempo_chegada: 0 },
        Processo { nome: "P2".into(), tempo_restante: 4,  tempo_chegada: 0 },
        Processo { nome: "P3".into(), tempo_restante: 9,  tempo_chegada: 0 },
        Processo { nome: "P4".into(), tempo_restante: 5,  tempo_chegada: 0 },
    ];

    round_robin(processos, 3);
    // P1: t=3, P2: t=7 (concluído), P3: t=10, P4: t=13, P1: t=16, P3: t=19,
    // P4: t=21 (concluído), P1: t=24 (concluído), P3: t=27 (concluído)
}
```

---

## Sumário de Complexidades

| # | Exercício | Tempo | Espaço |
|---|-----------|-------|--------|
| 1 | Inversão com Vec | O(n) | O(n) |
| 2 | Contador de ocorrências | O(n) | O(k) |
| 3 | Remoção condicional | O(n) | O(n) |
| 4 | Mescla ordenada (manual) | O(n+m) | O(n+m) |
| 5 | Calculadora RPN | O(n) | O(n) |
| 6 | Histórico de navegação | O(1)/op | O(n) |
| 7 | Desfazer/Refazer | O(1) amort. | O(n) |
| 8 | Símbolos balanceados | O(n) | O(n) |
| 9 | StackMin | O(1)/op | O(n) |
| 10 | Simulador fila banco | O(n) | O(n) |
| 11 | Impressora compartilhada | O(n) | O(n) |
| 12 | Buffer circular (overwrite) | O(1)/op | O(cap) |
| 13 | Fila prioridade manual | enq O(1) · deq O(n) | O(n) |
| 14 | Palíndromo com Deque | O(n) | O(n) |
| 15 | Janela deslizante máxima | O(n) | O(k) |
| 16 | Fila tarefas frente/fundo | O(1)/op | O(n) |
| 17 | Comparação desempenho | — | — |
| 18 | Quando usar qual TAD? | — | — |
| 19 | processar_em_lotes | O(n) | O(lote) |
| 20 | Round Robin | O(n·⌈t/Q⌉) | O(n) |
