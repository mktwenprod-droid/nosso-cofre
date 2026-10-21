# DuoVita

Cofre de poupança para casais: cada um entra com seu próprio login, registra os aportes, acompanha o saldo com rendimento estimado (estilo Nubank, ligado ao CDI) e simula quanto vão ter guardado até o casamento ou a entrada da casa.

> O repositório e a URL do GitHub Pages continuam se chamando `nosso-cofre` (mudar isso trocaria o link do site); o nome da marca dentro do site é **DuoVita**.

É um site estático (HTML/CSS/JS puro, sem build), sincronizado em tempo real entre dispositivos via **Firebase Firestore**, publicado no **GitHub Pages**.

## Configurar o Firebase (obrigatório para sincronizar)

1. Acesse [console.firebase.google.com](https://console.firebase.google.com) e crie um projeto novo (pode ser gratuito, plano Spark). Recomendado: um projeto **separado** do usado em outros sites seus, já que aqui ficam dados financeiros do casal.
2. No menu lateral, vá em **Firestore Database** → **Criar banco de dados** → escolha uma localização → modo **produção**.
3. Vá em **Authentication** → aba **Sign-in method** → ative o provedor **E-mail/senha**.
4. Vá em **Configurações do projeto** (ícone de engrenagem) → **Seus apps** → clique no ícone `</>` (Web) para registrar um app.
5. Copie o objeto `firebaseConfig` que aparece.
6. Abra o [index.html](index.html) deste projeto, procure o bloco `CONFIGURAÇÃO DO FIREBASE` (perto do início do `<script>`) e substitua os valores de exemplo pelos que você copiou.
7. Em **Firestore Database → Regras**, cole as regras abaixo. Elas liberam leitura e escrita só para quem estiver logado (não existe visão pública aqui — são dados financeiros só do casal):

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /nosso-cofre/{docId} {
      allow read, write: if request.auth != null;
    }
  }
}
```

8. Publique as regras (**Publicar**).
9. Faça commit e push da alteração no `index.html` (com suas chaves reais) para o GitHub — veja a seção "Publicar" abaixo.

> A `apiKey` do Firebase para apps Web não é um segredo (ela só identifica o projeto); quem protege os dados de verdade são as regras do Firestore acima, por isso é normal ela ficar visível no código publicado.

## Criar o login de cada um do casal

Não existe cadastro público — vocês dois precisam ser criados manualmente no Firebase:

1. No console do Firebase, vá em **Authentication → Users → Add user**.
2. Preencha um e-mail e uma senha para a primeira pessoa e clique em **Add user**.
3. Repita para a segunda pessoa, com outro e-mail.
4. Abra o site e entre com um desses logins — quem entrar primeiro faz um cadastro rápido (nome de cada um, meta opcional) e vira automaticamente "pessoa 1". Quando a segunda pessoa entrar com o próprio login pela primeira vez, o site pergunta "quem é você?" para vincular aquele e-mail ao nome certo.

Cada um só precisa fazer esse vínculo uma vez; depois disso o site já reconhece quem está logado. Dá pra trocar o vínculo depois clicando em "não é você?" no topo da tela.

## O que o site guarda

- **Aportes**: valor, data, quem aportou e uma nota opcional, com histórico completo e total por pessoa.
- **Renda do casal**: salário de cada um, usado só para calcular a divisão proporcional de um aporte mensal combinado (ex: se um ganha o dobro do outro, sugere 2/3 e 1/3).
- **Parâmetros de investimento**: taxa do CDI atual e percentual do CDI do produto usado (100% no RDB padrão do Nubank, até 120% na Caixinha Turbo) — editável a qualquer momento, pois a taxa muda com a Selic.
- **Simulação**: projeta o saldo futuro dado um aporte mensal e um prazo, ou calcula o aporte mensal necessário para bater uma meta de valor numa data.

O saldo com rendimento é uma **estimativa**, calculada localmente a partir da taxa configurada — não é extrato de banco. Confirme sempre a taxa vigente no app do Nubank ou no Banco Central.

## Publicar / acessar de qualquer lugar (GitHub Pages)

Depois que o repositório for publicado no GitHub com Pages habilitado, o site fica disponível em:

```
https://<seu-usuario>.github.io/nosso-cofre/
```

Qualquer atualização enviada (`git push`) para a branch `main` atualiza o site automaticamente em alguns minutos.

## Rodar localmente

Basta abrir o `index.html` no navegador (ou servir a pasta com qualquer servidor estático). Não precisa de build nem instalação — é só HTML/CSS/JS puro.
