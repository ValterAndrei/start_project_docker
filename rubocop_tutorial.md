# Rubocop


1. Instalando gem rubocop

```
group :development do
  gem 'rubocop', require: false
end
```

2. Gerando o arquivo de configuração `.rubocop_todo.yml`

```
$ rubocop -R --auto-gen-config
```

3. Desabilitando verificações específicas `.rubocop_todo.yml`

```
Style/TernaryParentheses:
  Exclude:
    - 'app/services/measurement_table.rb'

Style/FrozenStringLiteralComment:
  Exclude:
    - 'app/**/*'
    - 'bin/**/*'

Metrics/ClassLength:
  Enabled: false
  
```

4. Desabilitando todas as verificações usando o AllCops `.rubocop_todo.yml`

```
AllCops:
  Exclude:
    - 'app/**/*'
    - 'spec/**/*'
    - 'bin/**/*'
    - 'lib/**/*'
    - '**/*.rb'
    - '**/Gemfile'
    - 'node_modules/**/*'
    - 'app/services/measurement_table.rb'
```

5. Desabilitando verificações dentro do arquivo

```
# rubocop:disable all
# rubocop:disable Metrics/LineLength, Style/StringLiterals
```

6. Desabilitando verificações na frente do método

```
def very_long_method # rubocop:disable Metrics/MethodLength
  ...
end
```

7. Adicionando verificação em um arquivo específico

```
Metrics/ClassLength:
  Include:
    - '**/*er.rb'
```

8. Executando rubocop para o Rails

```
$ rubocop --rails

ou adicionando no .rubocop_todo.yml

Rails:
  Enabled: true
```

9. Aplicando auto correção

```
$ rubocop --auto-correct file_name.rb

ou..

$ rubocop -a --only "Layout/AlignParameters" file_name.rb
```

10. Removendo avisos `Ignoring GEM because its extensions are not built`

```
$ gem pristine --all
```


[Documentação](https://rubocop.readthedocs.io/en/latest/basic_usage/)
