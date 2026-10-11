# Fluig Form Attachment

Plugin JQuery para auxiliar no tratamento de anexos em formulários
de processo.

A gestão dos anexos de processo no Fluig não é uma tarefa fácil. Além da aba anexos não
permitir ordenação, o Fluig não permite, de maneira simples, vincular os anexos a campos
do formulário do processo, assim devemos confiar que os usuários vão colocar os anexos
corretamente na aba anexos e depois gastar tempo identificando cada anexo, pois usuários
dificilmente respeitarão alguma regra de nomenclatura.

Para permitir esse vínculo entre os campos do formulário e os anexos, o
[Sérgio Machado](https://www.linkedin.com/in/sergio-machado-analista-fluig/)
criou o [ComponenteAnexos](https://github.com/sergiomachadosilva/fluig-utils/tree/main/projetos/ComponenteAnexos),
que é uma biblioteca JS/CSS/HTML, permitindo um maior controle dos
anexos. Esse projeto do Sérgio serviu como base para a construção desse Plugin JQuery.

A intenção desse plugin é simplificar ainda mais o tratamento dos anexos em formulários
que estão abertos em um Processo.

## Atenção

Esse plugin não funciona na versão mobile dentro do aplicativo My Fluig.

Testado no Fluig 1.8.1 e 1.8.2. Quando lançar o Fluig 2.0 teremos que
rever o funcionamento do plugin.

Esse plugin está usando somente a "descrição" do anexo como vínculo entre
formulário e anexo. Ainda está em testes se isso é o suficiente ou se
o ideal seria ter o nome físico do arquivo salvo em algum lugar, assim
como o ComponenteAnexos do Sérgio Machado.

No Fluig 1.8.1 identifiquei um bug quando insere um anexo por vez e então remove
algum anexo. Nessa situação a tabela de anexos é esvaziada. Porém esse bug ocorre
mesmo quando usando a Aba de Anexos, sendo um bug do próprio Fluig e não do Plugin.
Esse bug ocorre somente na primeira atividade do processo, quando ainda não há ID
da Solicitação.

## Instalação

Basta adicionar o script `fluigFormAttachment.js` ou `fluigformAttachment.min.js` ao
seu formulário.

## Modo de usar

Ao carregar o plugin no seu formulário ele ocultará a Aba Anexos do Fluig, evitando que
insiram documentos não solicitados e, mais importante, removam anexos que já estão
vinculados a um campo do formulário.

Para controlar a visibilidade da aba de anexos use os métodos:
- `$.fluigFormAttachment.showAttachmentTab()`
- `$.fluigFormAttachment.hideAttachmentTab()`

### Básico

Basta ter um campo do tipo texto e instanciar o plugin para esse campo.

```html
<div class="form-group">
    <label for="cnh">Teste 001</label>
    <input type="text" class="form-control" readonly
        name="cnh" id="cnh" data-filename="CNH" data-accept="image/*,.pdf"
    >
</div>
```

É recomendável deixar o campo como `readonly`. O plugin fará isso automaticamente quando
instanciado no elemento, mas pode ser que algum usuário consiga editar o formulário fora
do processo e nesse caso o plugin não funcionará.

O atributo `data-filename` indica a "descrição" do anexo, que é o valor que aparece na Aba
Anexos. Esse nome será utilizado para vincular o campo do formulário ao anexo.

O atributo `data-accept` indica o tipo de arquivo permitido. Funciona exatamente como
o atributo `accept` do campo input file ([documentação](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes/accept)).

Então no JavaScript basta executar:

```javascript
$("#cnh").fluigFormAttachment();
```

### Parâmetros

Ao instanciar o plugin para os elementos é possível passar um objeto com as seguintes
propriedades de configuração:

| Parâmetro | Tipo | Padrão | Descrição |
| --- | --- | --- | --- |
| **showActionButton** | boolean | `true` | Indica se deve exibir os botões de Ação (Upload ou Delete) |
| **filename** | string | `"Anexo"` | Nome/Descrição do Anexo. Caso exista o atributo `data-filename` no campo do formulário ele terá preferência. **Cuidado**: Não pode ter mais de um anexo com o mesmo Nome/Descrição. |
| **prefixName** | boolean\|string | `false` | Adiciona prefixo à descrição do anexo. Caso `true` criará um prefixo aleatório usando parte de um UUID. No caso de `string` a usará como prefixo fixo, adicionando `-` como separador. |
| **accept** | string | `"*"` | Funciona igual ao atributo accept do input file ([documentação](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes/accept)). Caso exista o atributo `data-accept` no campo do formulário ele terá preferência. Lembre-se que isso não impede o usuário de inserir arquivos diferentes do indicado no accpet. |
| **validateUpload** | callback | `null` | Função, que recebe um `File` como parâmetro, para executar a validação do arquivo antes do upload. Em caso de arquivo inválido dispare uma exceção com string, isso impedirá o upload e mostrará o erro na barra de progresso do envio do anexo.|

Se ao instanciar o Plugin o campo estiver **Desabilitado** ou o usuário não possuir permissão
de complemento no processo, o botão de Upload/Delete não será exibido, mesmo se o parâmetro
`showActionButton` estiver como `true`. Caso deseje exibir/ocultar o botão de ação dinamicamente
utilize os métodos `showActionButton` e `hideActionButton`.

A intenção da configuração `prefixName` é auxiliar nos casos de Tabela Pai Filho.

Lembre-se de tratar as situações nas quais o processo estiver somente em modo de
visualização e nos casos que os campos não devem permitir upload/delete dos
anexos.

Exemplos:

```javascript
$("#cnh").fluigFormAttachment({
    showActionButton: false,
});

// Exibe os botões de visualização em todos os filhos já existentes da Pai Filho
// Pulamos a primeira TR por ser a base para gerar as demais filhas
$("#tabelaPaiFilho tbody tr:not(:first-child) .anexo")
    .fluigFormAttachment({ showActionButton: false })
;

// Tabela Pai Filho - adiciona com prefixo automático para cada arquivo
$("#adicionar").on("click", function () {
    const index = wdkAddChild("tabelaPaiFilho");
    $(`#cnh___${index}`).fluigFormAttachment({
        prefixName: true,
    });
});

// Tabela Pai Filho com vários campos de anexo
$("#adicionar").on("click", function () {
    const index = wdkAddChild("tabelaPaiFilho");
    const tr = $(`#cnh___${index}`).closest("tr");

    // A cada upload será criado um prefixo aleatório pra cada anexo
    $(".anexos", tr).fluigFormAttachment({
        prefixName: true,
    });
});

// Tabela Pai Filho com vários campos de anexo e prefixo único pra linha
$("#adicionar").on("click", function () {
    const index = wdkAddChild("tabelaPaiFilho");
    const tr = $(`#cnh___${index}`).closest("tr");

    // A cada upload usará o mesmo prefixo para todos os anexos da linha
    $(".anexos", tr).fluigFormAttachment({
        prefixName: FLUIGC.utilities.randomUUID().substring(0, 8),
    });
});
```

### Métodos

É possível executar alguns métodos do Plugin para manipular os anexos. Para
executar um método basta chamar o plugin no campo indicando passando o nome
do método ao invés do objeto de parâmetros.

Os seguintes métodos estão disponíveis:

| Método | Executa em | Parâmetros | Retorno | Descrição |
| --- | --- | --- | --- | --- |
| **hasAttachment** | Primeiro Elemento | - | `boolean` | Indica se o campo possuí anexo (tem descrição e o anexo está na tabela de anexos) |
| **isValid** | Primeiro Elemento | - | `boolean` | Indica se o campo está válido. Caso ele possua um valor (foi feito upload do anexo), mas o anexo não está na tabela de anexos, indicará campo inválido. Este método é executado em um único campo |
| **deleteAttachment** | Todos Elementos | - | `JQuery` | Remove o anexo do campo. Útil para quando excluir uma linha de uma tabela Pai Filho |
| **showActionButton** | Todos Elementos | - | `JQuery` | Exibe o botão de ação |
| **hideActionButton** | Todos Elementos | - | `JQuery` | Oculta o botão de ação |
| **prefixName** | Todos Elementos | string\|boolean | `JQuery` | Altera o prefixName para os elementos |
| **filename** | Todos Elementos | string, string\|boolean | `JQuery` | Altera o filename configurado para os elementos. Pode passar, como segundo parâmetro, o prefixName |

Exemplos:

```javascript
if ($("#cnh").fluigFormAttachment("hasAttachment")) {
    // Exemplo: habilitar outro campo de preenchimento
}

function beforeSendValidate(numState, nextState) {
    if (!$("#cnh").fluigFormAttachment("isValid")) {
        throw "O Anexo da CNH não foi enviado corretamente. Remova-o e envie novamente";
    }
}

// Removendo anexos da Pai Filho ao excluir linha
$("#tabelaPaiFilho").on("click", ".removeItem", function() {
    $(".anexos", $(this).closest("tr")).fluigFormAttachment("deleteAttachment");
    fnWdkRemoveChild(this);
});

// Habilitando/Desabilitando a ação de acordo com o preenchimento de um campo
$("#descricao").on("change", function () {
    this.value = this.value.trim();

    if (!this.value.length) {
        $("#anexo").fluigFormAttachment("hideActionButton");
        return;
    }

    // Trocando o filename pra ser dinâmico com a descrição
    // Caso exista uma descrição igual nos anexos exibirá erro
    // no momento de selecionar o arquivo.
    $("#anexo")
        .data("filename", this.value)
        .fluigFormAttachment("showActionButton")
    ;

    // Altera o filename igual ao exemplo anterior
    $("#anexo").fluigFormAttachment("filename", this.value).fluigFormAttachment("showActionButton");

    // Altera prefixName no componente
    $("#anexo").fluigFormAttachment("prefixName", true);
    // ou
    $("#anexo").fluigFormAttachment("prefixName", "novoprefixo");
});
```

## Contribuições

Sinta-se à vontade para indicar bugs e sugestões abrindo issues.
