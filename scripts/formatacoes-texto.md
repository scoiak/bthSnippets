#### Formatação de CPF/CNPJ

```groovy
def formataCpfCnpj = { in_campo ->
  campoFormatado = in_campo;
  tentar {
    se(in_campo.tamanho() == 11){
      campoFormatado = "${in_campo.subTexto(0,3)}.${in_campo.subTexto(3,3)}.\
${in_campo.subTexto(6,3)}-${in_campo.subTexto(9,2)}";
    }
    se(in_campo.tamanho() == 14){
      campoFormatado = "${in_campo.subTexto(0,2)}.${in_campo.subTexto(2,3)}.\
${in_campo.subTexto(5,3)}/${in_campo.subTexto(8,4)}-${in_campo.subTexto(12,2)}";
    }    
  } tratar {
    imprimir "Erro ao executar função 'formataCpfCnpj' para o valor [$in_campo]. #excecao";
  }
  retornar campoFormatado
}
```