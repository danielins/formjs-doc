# Spacelab Form.js (ECMAScript 2015/ES6)

Form.js é uma classe auxiliadora na configuração  em poucos passos de formulários validados e enviados via Spacelab WebForms. A classe utiliza jQuery e alguns métodos da Spacelab CoreUtils que necessitam de Backbone e Underscore (em especial, a ModalBox).


## Instalação

Arquivos necessários (caminho e nome dos arquivos podem variar de acordo com o projeto):

```html
<!-- BIBLIOTECAS -->
<script src="/website/v1/_js/_libs/jquery/jquery-1.12.0.min.js"></script>
<script src="/website/v1/_js/_libs/backbone/underscore.min.js"></script>
<script src="/website/v1/_js/_libs/backbone/backbone.min.js"></script>

<!-- SPACELAB WEBFORMS -->
<script src="/website/v1/_js/_libs/webforms/Web.Forms.min.js"></script>

<!-- SPACELAB COREUTIL -->
<script src="/website/v1/_js/_min/Core.min.js"></script>

<!-- SPACELAB FORM -->
<script src="/website/v1/_js/_libs/form/es6/_min/Form.min.js"></script>

<!-- CLASSE DO FORMULÁRIO -->
<script src="/website/v1/_js/_min/Contato.min.js"></script>
```


## Configuração básica

Para usar a Form.js, extendemos a classe base para uma classe para o formulário a ser validado:

```javascript
class Contato extends Form {

}
```

O método básico de configuração é o webforms_params, que manda os parâmetros de validação e envio para o WebForms. A classe base já possui boa parte dos parâmetros em seu valor padrão (validationType, errorClass, ajaxMethod, etc) então não é necessário configurá-los, exceto se for necessário alterar o valor padrão dos parâmetros.

```javascript
/** VALORES PADRÃO DOS PARÂMETROS DE CONFIGURAÇÃO DO WEBFORMS
 * Não é necesssário adicionar estes parâmetros se os valores forem os seguintes:
 */ 
 
 // tipo de validação do WebForms
 validationType = "normal";
 // classe de erro adicionada nos campos inválidos
 errorClass = "erro";
 // método de envio do AJAX
 ajaxMethod = "POST";
 // posicionamento dos alertas de erro
 alert = {
   type: 'outer',
   margin: false
 }
 
 // Os parâmetros onAjaxStart, on AjaxComplete e onClientSideError são atribuídos à métodos da classe base. 
 // O parâmetro form será o formulário à qual a classe foi instânciada (ver Iniciação).
 
```

Na maioria dos casos, só é necessário configurar o caminho do arquivo com as validações XML e caminho do arquivo para onde o formulário será enviado via AJAX:

```javascript
webforms_params(){

    // recupera os valores padrão
    let params = super.webforms_params();

    // adiciona configuração de validação (XML) e envio (AJAX)
    params.ajax = "/website/v1/_ajax/frm-contato.aspx?acao=enviar";
    params.xml = "/website/v1/_xml/frm-contato.xml";

    // retorna os parâmetros para a classe
    return params;

}
```

Resultado final:

```javascript
class Contato extends Form {
    webforms_params(){

        let params = super.webforms_params();

        params.ajax = "/website/v1/_ajax/frm-contato.aspx?acao=enviar";
        params.xml = "/website/v1/_xml/frm-contato.xml";

        return params;

    }
}
```

Com essa base (e os arquivos XML e AJAX devidamente configurados), o formulário está pronto para ser iniciado.

## Iniciação

Para iniciar a classe base instanciar a classe criada. A única informação que precisa ser passada para  a classe é para qual o <form> ela está sendo relacionada. Isso pode ser feito:

Via seletor:

```html
<form id="form-contato">
    ...
</form>
<script>
    var contato = new Contato({
        formSelector: "#form-contato"
    });
</script>
```

Via classe padrão:
```html
<form class="form-js">
    ...
</form>
<script>
    var contato = new Contato();
</script>
```

Importante: independe da forma de seleção, caso forem encotrados dois ou mais formulários, a classe irá se instanciar apenas no primeiro elemento encontrado. Para páginas com múltiplos formulários, o método via seletor é recomendado.


## Configurações de mensagem

O Form.js auxilia no recebimento e exibição de mensagens após o envio do formulário, em caso de sucesso ou erro de validação e/ou envio. Abaixo os parâmetros relacionados á exibição de mensagens: 

```javascript
/**
 * PARÂMETROS DE MENSAGEMS E SEUS VALORES PADRÃO
 */

// @configParam messageType
// {String} tipo da mensagem exibida quando o envio for concluido
// Obrigatoriamente precisa ser uma das opções:
// - modal: usa modal criado pelo CoreUtil.Modal
// - console: console.log()
// - alert: alert()
// - inline: mensagem aparece no topo do formulário
// - rewrite: o form é apagado e a mensagem aparece em seu lugar
// Se for colocado um valor diferente dos citados, será usado o valor padrão "modal"
messageType: "modal";

// @configParam rewriteOnFailure
// {String} se a mensagem deve apagar o formulário em caso de erro/falha
// caso o valor seja "rewrite" novamente, mensagens de erros irão sobrescrever o formulário
// caso contrário, dá uma segunda opção de tipo de mensagem
// !! configuração necessária apenas se o valor de messageType for "rewrite" !!
rewriteOnFailure: "modal";

// @configParam successMessage
// {String} mensagem de sucesso
successMessage: $("<p>Envio realizado com sucesso</p>").html();

// @configParam successCallback
// {Function} callback de successo
// -- É disparado quando houver sucesso no envio do formulário
successCallback: (mensagem = this.configParams.successMessage) => {
    this.showMessage(mensagem);
};

// @configParam failureMessage
// {String} mensagem de erro
failureMessage: $("<p>Ocorreu uma falha ao realizar o envio</p>").html();

// @configParam failureCallback
// {Function} callback de erro
// -- É disparado quando houver erro no envio do formulário
failureCallback: (mensagem = this.configParams.failureMessage) => {
    this.showMessage(mensagem);
};

// @confiParam invalidFieldMessage
// {String} mensagem padrão para quando há campos inválidos
invalidFieldMessage: $("<p>Existem campos inv&aacute;lidos. Revise-os e tente novamente.</p>").html();

```

A função `showMessage()` que é chamada pelas funções padrão de todos os callbacks exibe a mensagem (passada por parâmetro) de acordo com o tipo especificado no parâmetro messageType.

O padrão de mensagem via `jQuery.html()` é para assegurar caracteres especiais em diferentes encodings.

É possível personalizar os parâmetros na iniciação:

```html
<script>
  var contato = new Contato({
    formSelector: "#frm-contato",
    messageType: "rewrite",
    successMessage: $("<p>Sua mensagem foi enviada com sucesso.<br>Por favor, aguarde nosso retorno.</p>").html(),
    sucessCallback: function(){
      var message = this.configParams.successMessage;
      this.showMessage(message);
      // atualiza página após mensagem
      setTimeout(function(){
        location.reload();
      }, 10000);
    }
  });
</script>
```

Neste exemplo, a messagem de sucesso será exibida no corpo do elemento form, apagando os campos, e depois de 10 segundos a página será atualizada. Note que a função precisou ser escrita em formato ES5 pois o código está no HTML e não poderá ser compilado.


## Configuração após sucesso

As configurações abaixo descrevem o comportamento do formulário após um envio que não retornou erro.

```javascript
// @configParam resetOnSuccess
// {Boolean} se o form vai ser resetado no sucesso de envio
resetOnSuccess: true;

// @configParam showSubmitAfterSuccess
// {Boolean} se o botão de envio vai ser exibido de volta após o sucesso
showSubmitAfterSuccess: true;

// @configParam hideLoadingAfterSuccess
// {Boolean} se o loading vai ser escondido após o sucesos
// recomendado o false em caso de formulários que efetuam redirect no sucesso
hideLoadingAfterSuccess: true;
```

## Adicionando funcionalidades ao formulário

Caso seja necessário adicionar funcionalidades específicas no formulário ou página, é possível usá-las na função reservada `extend_config`, que é chamada durante a inicialização da classe. No exemplo abaixo, é aplicado o Spacelab Placeholder, uma máscara em um campo e é chamado um novo método `map()`, que foi declarado em seguida.

```javascript
class Contato extends Form {
 ...
 extend_config(){
  new Placeholder;

  $("#cmpTelefone").inputmask(["(99) 9999-9999", "(99) 99999-9999"]);
  
  this.map();
 }
 map(){
  ...
 }
}
```
