# Telegram_logs_Garoa_Hacker_Clube_Publico
Logs dos canais públicos de Telegram relatcionados ao Garoa Hacker Clube, dentre os quais o mais famoso é https://telegram.me/GaroaHC

Estes logs são, por enquanto, coletados manualmente pelo Juca e refletem o conteúdo dos canais sob a ótica deste usuário. As instruções abaixo são anotações para o próprio Juca. Provavelmente deve dar algum problema se o repositório for atualizado usando esses comandos por outro usuário. É algo que seria bom de se melhorar o script de dump (permitir uma coleta não centrada na perspectiva do usuário que executa o dump).

## procedimento de dump

Em um terminal:

```
cd ~/devel/github_vysheng/tg
bin/telegram-cli --json -P 9009
```

E em outro terminal:
```
cd ~/devel/github_garoa/Telegram_logs_Garoa_Hacker_Clube_Publico
ruby ~/devel/github_tvdstaaij/telegram-history-dump/telegram-history-dump.rb -c config-garoa.yaml
git add -A
git commit -m "mensagem"
push origin master
```
