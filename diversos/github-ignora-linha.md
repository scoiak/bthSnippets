### Github - Inognorar determinada linha no arquivo

O método abaixo detalha como configurar o github para ignorar apenas determinadas linhas de um arquivo ao realizar commit/push, e não o arquivo inteiro como o padrão.

Isso pode ser utilizado, por exemplo, para ocultar as linhas onde se passam senhas, tokens, dados pessoais ou credenciais em geral de forma literal no arquivo.

O procedimento é o seguinte:

- 1. No arquivo .gitignore, definir o filtro que deverá ser utilizado. No exemplo, é utilizado o filtro "ignoreline"

```bash
*.py filter=ignoreline
```

- 2. No terminal da aplicação, rodar o seguinte comando para informar ao git que essas linhas deverão ser ignoradas:
```bash
git config --global filter.ignoreline.clean "sed '/#ignoreline$/'d"
```

- 3. Por fim, executar o seguinte comando no terminal para aplicar a regra no pull
```bash
git config --global filter.ignoreline.smudge cat
```

- 4. Nos arquivos de código, ao adicionar o texto do filtro, no caso 'ignoreline', toda a linha será desconsiderada do commit/push. Segue exemplo de utilização:
```python
    params_execucao = {
        'clicodigo': '2016',
        'somente_pre_validar': False,
        'token': '72612895-xxxx-xxxx-xxxx-36b8b57c3198', #ignoreline
        'ano': 2020
    }
```

Se a configuração foi feita corretamente, o código acima será enviado ao repositório da seguinte forma:
```python
    params_execucao = {
        'clicodigo': '2016',
        'somente_pre_validar': False,
        'ano': 2020
    }
```