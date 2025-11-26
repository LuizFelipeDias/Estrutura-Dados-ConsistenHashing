<script setup>
const youtubeUrl = 'https://www.youtube.com/watch?v=thbd_usXyEg'
const youtubeEmbedUrl = 'https://www.youtube.com/embed/thbd_usXyEg'
</script>

<template>
  <div class="min-h-screen bg-white text-slate-900 flex items-center justify-center">
    <div class="max-w-6xl w-full px-4 py-8 space-y-8">

      <!-- TÍTULO -->
      <header class="space-y-2 text-center">
        <h1 class="text-2xl md:text-3xl font-bold">
          Consistent Hashing (Hash Consistente)
        </h1>
        <p class="text-sm text-slate-500 max-w-3xl mx-auto">
          Nesta página você vê uma visão conceitual do hash consistente e como o código
          em Python organiza nós e chaves em um anel lógico para reduzir o rebalanceamento
          quando servidores entram ou saem do cluster.
        </p>
      </header>

      <!-- VÍDEO DO YOUTUBE EM DESTAQUE -->
      <section class="space-y-3">
        <h2 class="text-lg font-semibold text-center">
          Vídeo de apoio
        </h2>

        <div class="max-w-3xl mx-auto space-y-3">
          <div class="aspect-video rounded-xl overflow-hidden shadow">
            <iframe
                :src="youtubeEmbedUrl"
                title="Consistent Hashing - vídeo de referência"
                frameborder="0"
                allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
                allowfullscreen
                class="w-full h-full"
            ></iframe>
          </div>

          <p class="text-xs text-center text-slate-500">
            Link direto:&nbsp;
            <a
                :href="youtubeUrl"
                target="_blank"
                rel="noopener noreferrer"
                class="underline hover:text-slate-700"
            >
              {{ youtubeUrl }}
            </a>
          </p>
        </div>
      </section>

      <!-- CONTEÚDO PRINCIPAL – TUDO EMPILHADO, UM BLOCO DEPOIS DO OUTRO -->
      <main class="space-y-10">

        <!-- BLOCO 1: EXPLICAÇÃO GERAL -->
        <section class="space-y-4 text-sm leading-relaxed">
          <h2 class="text-lg font-semibold">
            O que é hash consistente?
          </h2>

          <p>
            Em sistemas distribuídos, precisamos decidir <strong>em qual servidor</strong> uma chave
            (por exemplo, um usuário, sessão, pedido ou arquivo) será armazenada. Uma abordagem
            bem comum é usar uma fórmula do tipo <code>hash(chave) % N</code>, em que
            <code>N</code> é o número de servidores.
          </p>

          <p>
            Esse método funciona enquanto a quantidade de servidores permanece fixa.
            O problema aparece quando o cluster cresce ou encolhe: se <code>N</code> muda,
            praticamente <strong>todas as chaves são remapeadas</strong>, o que significa
            reconstruir cache, mover dados entre máquinas, aumentar o tráfego de rede
            e, muitas vezes, degradar a performance do sistema.
          </p>

          <p>
            O <strong>hash consistente</strong> foi criado exatamente para reduzir esse impacto.
            A ideia é representar o espaço de hash como um <strong>anel lógico</strong>
            (por exemplo, de 0&deg; a 359&deg;). Tanto os servidores quanto as chaves são
            mapeados para posições nesse anel:
          </p>

          <ul class="list-disc list-inside space-y-1">
            <li>
              cada servidor é colocado em um ou mais pontos do anel (nós físicos
              e nós virtuais);
            </li>
            <li>
              cada chave também recebe uma posição (ângulo) no mesmo anel;
            </li>
            <li>
              cada chave pertence ao <strong>primeiro servidor no sentido horário</strong>
              a partir da sua posição.
            </li>
          </ul>

          <p>
            Quando um novo servidor entra, ele apenas “rouba” o pequeno trecho de anel que
            está logo antes dele. Só as chaves que caem nesse intervalo mudam de lugar.
            Quando um servidor sai, apenas as chaves que estavam com ele são redistribuídas.
            O resultado prático é um <strong>rebalanceamento mínimo</strong>, mesmo em cenários
            com muitos nós entrando e saindo.
          </p>

          <p class="text-xs text-slate-500">
            A seguir, você verá como isso é implementado em Python, com classes simples que
            poderiam estar em um repositório real de back-end ou de sistemas distribuídos.
          </p>
        </section>

        <!-- BLOCO 2: NODE E RING BÁSICO -->
        <section class="space-y-4 text-sm leading-relaxed">
          <h2 class="text-lg font-semibold">
            Estruturas básicas: <code>Node</code> e <code>Ring</code>
          </h2>

          <p>
            No código, cada servidor é representado por um <code>Node</code>. Ele guarda:
          </p>

          <ul class="list-disc list-inside space-y-1">
            <li><code>data</code>: metadados do servidor (ex.: <code>{"name": "ServidorA"}</code>);</li>
            <li><code>ang</code>: posição (ângulo) do servidor no anel; </li>
            <li><code>keys</code>: lista de chaves pelas quais esse nó físico é responsável.</li>
          </ul>

          <p>
            A estrutura <code>Ring</code> é o “cérebro” do hash consistente: ela mantém
            uma lista ordenada com todos os nós (físicos e virtuais) e implementa
            as operações de adicionar, remover e localizar nós.
          </p>

          <div class="bg-slate-900 text-emerald-50 rounded-lg shadow border border-slate-800">
            <div class="flex items-center justify-between text-[11px] text-slate-400 px-4 py-2 border-b border-slate-800">
              <span>hash_ring.py</span>
              <span>Python · Estruturas principais</span>
            </div>
            <pre class="text-[11px] md:text-xs p-4 overflow-x-auto">
<code class="language-python">
import json


class Node:
    def __init__(self, data):
        """
        Nó físico ou virtual do anel.
        data: dict com pelo menos {"name": "..."}.
        ang: posição (0-359) no anel lógico.
        keys: lista de chaves sob responsabilidade desse nó físico.
        """
        self.data = data
        self.ang = None
        self.keys = []

    def __repr__(self):
        return f"{self.data['name']}:{self.ang} -&gt; {len(self.keys)} keys"


class Ring:
    def __init__(self, replicas=50):
        """
        Estrutura principal do hash consistente.
        ring: lista de nós (físicos + virtuais) ordenados pelo ângulo.
        replicas: quantidade de nós virtuais por nó físico.
        """
        self.ring = []
        self.replicas = replicas
</code>
            </pre>
          </div>

          <p class="text-xs text-slate-500">
            Os nós virtuais são criados a partir do mesmo servidor físico, mas com ângulos
            diferentes, o que ajuda a suavizar a distribuição de chaves e evitar que um nó
            fique com uma fatia desproporcional do anel apenas por “sorte” do hash.
          </p>
        </section>

        <!-- BLOCO 3: FUNÇÕES DE HASH -->
        <section class="space-y-4 text-sm leading-relaxed">
          <h2 class="text-lg font-semibold">
            Hash de nós e hash de chaves
          </h2>

          <p>
            Para posicionar nós e chaves no anel, o código usa duas funções de hash:
          </p>

          <ul class="list-disc list-inside space-y-1">
            <li>
              <code>_hash_node</code>: recebe um nó (físico ou virtual) e devolve um ângulo
              no anel;
            </li>
            <li>
              <code>_hash_key</code>: recebe uma chave (dict ou string) e devolve o ângulo
              correspondente.
            </li>
          </ul>

          <p>
            Um detalhe importante é a preocupação em transformar tudo em
            <strong>string canônica</strong> (JSON ordenado). Isso garante que a mesma chave
            ou o mesmo servidor sempre gerem o mesmo hash, independentemente da ordem
            dos campos no dicionário, por exemplo.
          </p>

          <div class="bg-slate-900 text-emerald-50 rounded-lg shadow border border-slate-800">
            <div class="flex items-center justify-between text-[11px] text-slate-400 px-4 py-2 border-b border-slate-800">
              <span>hash_ring.py</span>
              <span>Python · Funções de hash</span>
            </div>
            <pre class="text-[11px] md:text-xs p-4 overflow-x-auto">
<code class="language-python">
class Ring:
    ...

    def _hash_node(self, node_data):
        """
        - Se node_data é um dict, converte para JSON ordenado.
        - Se é string (ex.: "ServidorA:1"), usa a string diretamente.
        Retorna um inteiro entre 0 e 359.
        """
        if isinstance(node_data, dict):
            raw = json.dumps(node_data, sort_keys=True)
        else:
            raw = node_data
        return abs(hash(raw)) % 360

    def _hash_key(self, key):
        """
        Faz hash da representação em string da chave.
        Isso garante que a mesma chave sempre vá para a mesma
        posição no anel, enquanto a topologia não muda.
        """
        if isinstance(key, dict):
            key_str = json.dumps(key, sort_keys=True)
        else:
            key_str = key
        return abs(hash(key_str)) % 360
</code>
            </pre>
          </div>

          <p class="text-xs text-slate-500">
            Transformar tudo em string antes de hashear é uma prática simples que melhora
            muito a previsibilidade do sistema, principalmente quando as chaves são estruturas
            mais ricas (como dicionários com vários campos).
          </p>
        </section>

        <!-- BLOCO 4: ADD / REMOVE E REBALANCEAMENTO -->
        <section class="space-y-4 text-sm leading-relaxed">
          <h2 class="text-lg font-semibold">
            Adicionando e removendo nós com rebalanceamento mínimo
          </h2>

          <p>
            Uma das grandes vantagens do hash consistente é que a entrada ou saída
            de um servidor impacta apenas uma fração das chaves. O código captura essa ideia
            nos métodos <code>add</code> e <code>remove</code>.
          </p>

          <p>
            Ao chamar <code>add</code>, o anel:
          </p>

          <ul class="list-disc list-inside space-y-1">
            <li>cria o nó físico correspondente ao servidor;</li>
            <li>cria várias réplicas virtuais com nomes como <code>"ServidorA:0"</code>, <code>"ServidorA:1"</code>, etc.;</li>
            <li>insere todos esses nós na lista <code>ring</code> e a ordena por ângulo;</li>
            <li>
              identifica, para cada novo nó, quais chaves que antes pertenciam ao sucessor
              agora devem ficar com ele;
            </li>
            <li>
              envia apenas esse conjunto de chaves para <code>_rebalance_keys</code>, que faz a
              redistribuição mínima.
            </li>
          </ul>

          <p>
            Quando um servidor é removido via <code>remove</code>, o processo é o oposto:
            coletam-se as chaves que estavam naquele nó físico e elas são redistribuídas
            para os demais nós do anel.
          </p>

          <div class="bg-slate-900 text-emerald-50 rounded-lg shadow border border-slate-800">
            <div class="flex items-center justify-between text-[11px] text-slate-400 px-4 py-2 border-b border-slate-800">
              <span>hash_ring.py</span>
              <span>Python · add / remove</span>
            </div>
            <pre class="text-[11px] md:text-xs p-4 overflow-x-auto">
<code class="language-python">
class Ring:
    ...

    def add(self, data):
        """Adiciona um nó físico e réplicas virtuais ao anel."""
        node = Node(data)
        node.ang = self._hash_node(node.data)

        new_nodes = [node]
        for i in range(self.replicas):
            rep_key = f"{data['name']}:{i}"
            replica = Node(data)
            replica.ang = self._hash_node(rep_key)
            new_nodes.append(replica)

        self.ring.extend(new_nodes)
        self.ring.sort(key=lambda n: n.ang)

        # Determina quais chaves cruzam o limite do novo nó
        keys_to_move = set()
        # (lógica completa foi omitida aqui para foco didático)
        ...

        if keys_to_move:
            self._rebalance_keys(list(keys_to_move))

    def remove(self, data):
        """Remove o nó físico e suas réplicas, redistribuindo apenas as chaves órfãs."""
        removed_keys = []
        physical = self.get_physical_node(data)
        if physical:
            removed_keys = physical.keys

        self.ring = [n for n in self.ring if n.data['name'] != data['name']]

        if removed_keys:
            self._rebalance_keys(removed_keys)
</code>
            </pre>
          </div>

          <p class="text-xs text-slate-500">
            Na prática, isso significa que a adição de um novo servidor é “barata”: apenas as chaves
            que caem na fatia do anel daquele nó é que precisam mudar de posição, preservando a
            estabilidade do cluster.
          </p>
        </section>

        <!-- BLOCO 5: REBALANCEAMENTO INTERNO -->
        <section class="space-y-4 text-sm leading-relaxed">
          <h2 class="text-lg font-semibold">
            Rebalanceamento interno: <code>_rebalance_keys</code>
          </h2>

          <p>
            O método <code>_rebalance_keys</code> é o motor interno que realmente remove
            e reatribui chaves entre os nós físicos. Ele pode operar em dois modos:
          </p>

          <ul class="list-disc list-inside space-y-1">
            <li>
              <strong>Full rebalance</strong> (quando <code>keys_to_assign</code> é <code>None</code>):
              limpa todas as chaves de todos os nós e redistribui tudo do zero;
            </li>
            <li>
              <strong>Minimum rebalance</strong> (quando recebe uma lista de chaves):
              remove apenas essas chaves dos nós físicos e as reatribui aos novos donos.
            </li>
          </ul>

          <p>
            O modo mínimo é o que nos interessa para manter o sistema estável: ele é usado
            por <code>add</code>, <code>remove</code> e <code>assign_keys</code>, sempre
            operando em cima de um subconjunto de chaves que precisam mudar de lugar.
          </p>

          <div class="bg-slate-900 text-emerald-50 rounded-lg shadow border border-slate-800">
            <div class="flex items-center justify-between text-[11px] text-slate-400 px-4 py-2 border-b border-slate-800">
              <span>hash_ring.py</span>
              <span>Python · _rebalance_keys()</span>
            </div>
            <pre class="text-[11px] md:text-xs p-4 overflow-x-auto">
<code class="language-python">
class Ring:
    ...

    def _rebalance_keys(self, keys_to_assign=None):
        """
        Motor central de atribuição e reatribuição de chaves.
        Sempre trabalha com strings JSON internamente.
        """
        keys_to_reassign_strings = []

        if keys_to_assign is None:
            # FULL rebalance (pega todas as chaves de todos os nós)
            for n in self.ring:
                keys_to_reassign_strings.extend(n.keys)
                n.keys.clear()
        else:
            # MINIMUM rebalance (apenas as chaves passadas em keys_to_assign)
            keys_to_remove_set = set()
            for key_item in keys_to_assign:
                if isinstance(key_item, dict):
                    keys_to_remove_set.add(json.dumps(key_item, sort_keys=True))
                else:
                    keys_to_remove_set.add(key_item)

            keys_to_reassign_strings = list(keys_to_remove_set)

            # Remove essas chaves dos nós físicos
            for n in self.ring:
                if n.ang != self._hash_node(n.data):
                    continue
                if n.keys:
                    n.keys = [
                        k_str for k_str in n.keys
                        if k_str not in keys_to_remove_set
                    ]

        # Fase comum: reatribui cada chave ao seu novo nó
        for key_str in keys_to_reassign_strings:
            node = self.get_node(key_str)
            if node:
                node.keys.append(key_str)
</code>
            </pre>
          </div>

          <p class="text-xs text-slate-500">
            Essa função centraliza toda a lógica de movimentação de chaves. Ao manter o trabalho
            restrito a uma lista pequena de chaves, ela garante que o custo de adaptação a mudanças
            de topologia seja controlado.
          </p>
        </section>

        <!-- BLOCO 6: LOOKUP, ATRIBUIÇÃO E SIMULAÇÃO -->
        <section class="space-y-4 text-sm leading-relaxed">
          <h2 class="text-lg font-semibold">
            Localizando o nó de uma chave e simulando a distribuição
          </h2>

          <p>
            Depois que o anel está montado, precisamos de uma forma rápida de descobrir
            <strong>quem é o dono</strong> de uma determinada chave. Essa responsabilidade
            é do método <code>get_node</code>:
          </p>

          <ol class="list-decimal list-inside space-y-1">
            <li>calcula o ângulo da chave com <code>_hash_key</code>;</li>
            <li>percorre a lista ordenada de nós até achar o primeiro com <code>ang &gt;= ang_da_chave</code>;</li>
            <li>se chegar ao fim sem encontrar, “dá a volta” e usa o primeiro nó do anel.</li>
          </ol>

          <p>
            A função <code>main</code> demonstra o fluxo completo de uma simulação:
            leitura de arquivos JSON, montagem do anel, distribuição inicial das chaves,
            adição e remoção de servidores e impressão da carga final.
          </p>

          <div class="bg-slate-900 text-emerald-50 rounded-lg shadow border border-slate-800">
            <div class="flex items-center justify-between text-[11px] text-slate-400 px-4 py-2 border-b border-slate-800">
              <span>hash_ring.py</span>
              <span>Python · lookup e main()</span>
            </div>
            <pre class="text-[11px] md:text-xs p-4 overflow-x-auto">
<code class="language-python">
class Ring:
    ...

    def get_node(self, key):
        """Retorna o nó físico responsável por uma determinada chave."""
        if not self.ring:
            return None

        key_ang = self._hash_key(key)

        for node in self.ring:
            if node.ang >= key_ang:
                return self.get_physical_node(node.data)

        # se a chave 'passa' do fim do anel, volta para o primeiro nó
        return self.get_physical_node(self.ring[0].data)


def read_json_file(file_path):
    with open(file_path, "r", encoding="utf-8") as f:
        return json.load(f)


def main():
    nodes = read_json_file("nodes.json")  # ex.: [{"name": "ServidorA"}, ...]
    keys = read_json_file("keys.json")    # lista de chaves (dicts ou strings)

    ring = Ring(replicas=50)

    # adiciona servidores iniciais
    for node in nodes:
        print(f"Adicionando {node['name']}...")
        ring.add(node)

    # distribuição inicial
    print("\nAtribuindo chaves...")
    ring.assign_keys(keys)
    print("Distribuição inicial:")
    ring.print_load()
    print("\n---\n")

    # mudanças de topologia
    print("Adicionando ServidorD...")
    ring.add({"name": "ServidorD"})
    print("\nRemovendo ServidorB...")
    ring.remove({"name": "ServidorB"})

    print("\nDistribuição após alterações:")
    ring.print_load()
    print("\n---\n")
</code>
            </pre>
          </div>

          <p class="text-xs text-slate-500">
            Esse tipo de simulação é útil para validar se a distribuição está razoavelmente uniforme
            e para medir, em números, quantas chaves migram quando novos servidores entram ou saem
            do sistema.
          </p>
        </section>

        <!-- BLOCO 7: EXEMPLOS PEQUENOS EM PYTHON -->
        <section class="space-y-4 text-sm leading-relaxed">
          <h2 class="text-lg font-semibold">
            Exemplos pequenos de uso em Python
          </h2>

          <p>
            Para testar o comportamento do anel sem depender de arquivos externos,
            podemos criar exemplos mínimos que simulam alguns cenários típicos:
          </p>

          <p class="font-semibold">
            Exemplo 1 – Distribuição inicial de chaves em três servidores
          </p>

          <div class="bg-slate-900 text-emerald-50 rounded-lg shadow border border-slate-800">
            <div class="flex items-center justify-between text-[11px] text-slate-400 px-4 py-2 border-b border-slate-800">
              <span>exemplo_minimo.py</span>
              <span>Python · Uso básico</span>
            </div>
            <pre class="text-[11px] md:text-xs p-4 overflow-x-auto">
<code class="language-python">
from hash_ring import Ring  # supondo que o código está em hash_ring.py

def exemplo_basico():
    ring = Ring(replicas=10)

    # 1) Adiciona três servidores
    ring.add({"name": "ServidorA"})
    ring.add({"name": "ServidorB"})
    ring.add({"name": "ServidorC"})

    # 2) Define algumas chaves simples
    keys = [
        "user:1",
        "user:2",
        "user:3",
        "user:4",
        "sessao:abc",
        "sessao:def",
    ]

    # 3) Atribui essas chaves
    ring.assign_keys(keys)

    print("Distribuição inicial de chaves:")
    ring.print_load()


if __name__ == "__main__":
    exemplo_basico()
</code>
            </pre>
          </div>

          <p class="font-semibold">
            Exemplo 2 – Ver em qual nó cada chave caiu
          </p>

          <div class="bg-slate-900 text-emerald-50 rounded-lg shadow border border-slate-800">
            <div class="flex items-center justify-between text-[11px] text-slate-400 px-4 py-2 border-b border-slate-800">
              <span>exemplo_lookup.py</span>
              <span>Python · Consulta de dono da chave</span>
            </div>
            <pre class="text-[11px] md:text-xs p-4 overflow-x-auto">
<code class="language-python">
from hash_ring import Ring

def exemplo_lookup():
    ring = Ring(replicas=5)
    ring.add({"name": "ServidorA"})
    ring.add({"name": "ServidorB"})

    keys = ["user:1", "user:2", "user:3"]
    ring.assign_keys(keys)

    for k in keys:
        node = ring.get_node(k)
        print(f"Chave {k} está no nó {node.data['name']} (ângulo {node.ang})")


if __name__ == "__main__":
    exemplo_lookup()
</code>
            </pre>
          </div>

          <p class="font-semibold">
            Exemplo 3 – Adicionando um servidor novo e observando a mudança
          </p>

          <div class="bg-slate-900 text-emerald-50 rounded-lg shadow border border-slate-800">
            <div class="flex items-center justify-between text-[11px] text-slate-400 px-4 py-2 border-b border-slate-800">
              <span>exemplo_adicao.py</span>
              <span>Python · Efeito de adicionar nó</span>
            </div>
            <pre class="text-[11px] md:text-xs p-4 overflow-x-auto">
<code class="language-python">
from hash_ring import Ring

def exemplo_adicao():
    ring = Ring(replicas=10)
    ring.add({"name": "ServidorA"})
    ring.add({"name": "ServidorB"})

    keys = [f"user:{i}" for i in range(1, 11)]
    ring.assign_keys(keys)

    print("Antes de adicionar ServidorC:")
    ring.print_load()

    print("\nAdicionando ServidorC...")
    ring.add({"name": "ServidorC"})

    print("\nDepois de adicionar ServidorC:")
    ring.print_load()


if __name__ == "__main__":
    exemplo_adicao()
</code>
            </pre>
          </div>

          <p class="text-xs text-slate-500">
            Esses exemplos pequenos ajudam a visualizar na prática como o anel reage a
            mudanças de topologia e como o conceito de hash consistente reduz o número
            de chaves que precisam ser movidas quando o cluster é alterado.
          </p>
        </section>
      </main>

      <!-- RODAPÉ / FONTES -->
      <footer class="mt-4 text-xs text-slate-500 space-y-1">
        <p>
          Conceitos de hash consistente amplamente utilizados em sistemas distribuídos
          modernos, como caches e bancos particionados, inspirados em implementações
          acadêmicas e industriais.
        </p>
      </footer>
    </div>
  </div>
</template>
