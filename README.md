# ☁️ Serverless Data Processing & Reporting with AWS Lambda

> ℹ️ **NOTE:** Este é o repositório desenvolvido por **Ismael Santos de Medeiros**.

Projeto com o objetivo de implantar e configurar uma arquitetura sem servidor (Serverless) orientada a eventos na AWS para automação de relatórios analíticos de vendas. A solução utiliza funções AWS Lambda encadeadas, AWS Systems Manager Parameter Store para gestão de credenciais, camadas customizadas (Lambda Layers), Amazon SNS para notificações por e-mail e gatilhos temporizados via Amazon EventBridge (CloudWatch Events).

A solução conecta-se de forma segura a um banco de dados MySQL (`cafe_db`) executado em uma instância Amazon EC2 (pilha LAMP) dentro de uma Amazon VPC pública, aplicando regras de Security Groups e trocas de mensagens assíncronas.

---

## 💻 Tecnologias utilizadas no projeto

- [AWS Lambda](https://aws.amazon.com/lambda/) — Execução de código Serverless (funções `salesAnalysisReport` e `salesAnalysisReportDataExtractor`)
- [Lambda Layers](https://docs.aws.amazon.com/lambda/latest/dg/configuration-layers.html) — Empacotamento e reuso da biblioteca cliente `PyMySQL` para Python 3.9
- [AWS Systems Manager (SSM)](https://aws.amazon.com/systems-manager/) — Armazenamento seguro de parâmetros de conexão com o banco de dados (`Parameter Store`)
- [Amazon SNS](https://aws.amazon.com/sns/) — Publicação e assinatura de tópicos de e-mail para envio de relatórios
- [Amazon EventBridge / CloudWatch Events](https://aws.amazon.com/eventbridge/) — Agendamento de execuções automatizadas via expressões Cron
- [AWS IAM](https://aws.amazon.com/iam/) — Configuração de papéis de execução com permissões para VPC, CloudWatch Logs, SNS e invocação de funções Lambda
- [Amazon EC2 & VPC](https://aws.amazon.com/ec2/) — Hospedagem do banco de dados MySQL da cafeteria e integração com interface de rede elástica (ENI)

---

## ✨ Como foi feito ?

- Análise e validação dos perfis de execução do IAM (`salesAnalysisReportRole` e `salesAnalysisReportDERole`) garantindo o acesso aos logs, VPC e invocação cruzada de funções.
- Criação de uma camada Lambda customizada (`pymysqlLibrary`) com a biblioteca `PyMySQL` para reuso entre funções Python.
- Implantação da função extratora `salesAnalysisReportDataExtractor` e vinculação à camada e à VPC da aplicação.
- Ajuste das regras de entrada do Grupo de Segurança (Security Group) da instância EC2 liberando a porta `3306` (MySQL) após diagnóstico de timeout nos testes iniciais.
- Geração de dados de teste realizando pedidos reais na aplicação web do Café hospedada na instância EC2.
- Criação e confirmação de assinatura de um Tópico no Amazon SNS (`salesAnalysisReportTopic`) para distribuição do relatório final por e-mail.
- Implantação da função orquestradora `salesAnalysisReport` via AWS CLI e configuração da variável de ambiente `topicARN`.
- Configuração de gatilho diário temporizado no EventBridge usando expressões Cron para disparo automático do relatório.

---

## 📚 Materiais

- Camada da biblioteca PyMySQL: `pymysql-v3.zip`
- Código da função extratora: `salesAnalysisReportDataExtractor-v3.zip`
- Código da função principal: `salesAnalysisReport-v2.zip`
- Documentação AWS Lambda: [AWS Lambda Developer Guide](https://docs.aws.amazon.com/lambda/)
- Documentação Amazon EventBridge Cron: [Schedule Expressions for Rules](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-create-scheduled-rule.html)

---

## 🛠️ Instruções de execução

Para replicar o resultado e implantar a arquitetura Serverless de relatórios, siga o passo a passo abaixo.

- 🤖 1. Crie a função extratora `salesAnalysisReportDataExtractor` no console do AWS Lambda.

![Criação da Função Extratora](1%20\(30\).png)

- 🤖 2. Crie a camada personalizada `pymysqlLibrary` com o arquivo `pymysql-v3.zip` e selecione-a na janela do Lambda.

![Seleção da Camada Lambda](1%20\(1\).png)

- 🤖 3. Confirme a adição da camada `pymysqlLibrary` na seção de configurações do runtime.

![Camada Vinculada](1%20\(2\).png)

- 🤖 4. Configure o manipulador da função como `salesAnalysisReportDataExtractor.lambda_handler`.

![Ajuste do Manipulador](1%20\(3\).png)

- 🤖 5. Importe o código Python da função extratora via arquivo `.zip` no editor do console.

![Código da Função Extratora](1%20\(4\).png)

- 🤖 6. Associe a função à `Cafe VPC`, à `Sub-rede pública 1` e ao grupo de segurança `CafeSecurityGroup`.

![Configuração de VPC e Security Group](1%20\(5\).png)

- 🤖 7. Valide a associação da VPC e sub-redes na página de visão geral da função.

![Visão Geral da Função na VPC](1%20\(6\).png)

- 🤖 8. Consulte a senha do banco de dados no Systems Manager Parameter Store no caminho `/cafe/dbPassword`.

![Consulta no Parameter Store](1%20\(7\).png)

- 🤖 9. Na guia de testes, configure um evento síncrono chamado `SARDETestEvent`.

![Configuração do Evento de Teste](1%20\(8\).png)

- 🤖 10. Insira o objeto JSON contendo os parâmetros do banco de dados (`dbUrl`, `dbName`, `dbUser`, `dbPassword`).

![JSON do Evento de Teste](1%20\(9\).png)

- 🤖 11. Execute o teste inicial para validar a conectividade com o banco de dados.

![Disparo do Teste da Função Extratora](1%20\(10\).png)

- 🤖 12. Acesse as configurações de VPC e adicione uma regra de entrada no `CafeSecurityGroup` liberando a porta `3306` (MySQL).

![Liberando Porta 3306 no Security Group](1%20\(11\).png)

- 🤖 13. Re-execute o teste da função extratora e confirme o sucesso na resposta da execução.

![Sucesso no Teste da Função Extratora](1%20\(12\).png)

- 🤖 14. Observe que o retorno do JSON inicial do corpo vem vazio (`body: []`) pois ainda não há pedidos no banco.

![Retorno do JSON Vazio](1%20\(13\).png)

- 🤖 15. Acesse o site da cafeteria usando o IP público da instância EC2.

![Navegação no Site da Cafeteria](1%20\(14\).png)

- 🤖 16. Realize pedidos de produtos no site para inserir registros reais no MySQL.

![Confirmação do Pedido na Aplicação Web](1%20\(15\).png)

- 🤖 17. Execute novamente o teste na função extratora e confirme a extração dos dados dos pedidos efetuados.

![Retorno com Dados de Vendas Extraídos](1%20\(16\).png)

- 🤖 18. No console do Amazon SNS, crie o tópico padrão `salesAnalysisReportTopic`.

![Criação do Tópico SNS](1%20\(17\).png)

- 🤖 19. Crie uma assinatura do tipo E-mail no tópico SNS e verifique o status inicial de confirmação pendente.

![Assinatura Pendente no SNS](1%20\(18\).png)

- 🤖 20. Acesse sua caixa de entrada e clique no link de confirmação enviado pela AWS.

![E-mail de Confirmação do SNS](1%20\(27\).png)

- 🤖 21. Verifique se o status da assinatura no SNS mudou para `Confirmado`.

![Assinatura Confirmada no SNS](1%20\(19\).png)

- 🤖 22. Conecte-se ao `CLI Host` via EC2 Instance Connect e configure suas credenciais com `aws configure`.

![Configuração da AWS CLI](1%20\(20\).png)

- 🤖 23. Crie a função orquestradora `salesAnalysisReport` executando o comando `aws lambda create-function` na CLI.

![Criação da Função via AWS CLI](1%20\(21\).png)

- 🤖 24. No console do Lambda, confirme que ambas as funções estão listadas e ativas.

![Lista de Funções Lambda](1%20\(22\).png)

- 🤖 25. Copie o ARN do tópico SNS criado anteriormente.

![Cópia do ARN do SNS](1%20\(23\).png)

- 🤖 26. Na função `salesAnalysisReport`, adicione a variável de ambiente `topicARN` com o ARN do tópico.

![Configuração da Variável topicARN](1%20\(24\).png)

- 🤖 27. Salve as alterações e confirme a atualização da função orquestradora.

![Variável de Ambiente Salva](1%20\(25\).png)

- 🤖 28. Execute um teste síncrono na função `salesAnalysisReport`.

![Teste da Função Orquestradora](1%20\(26\).png)

- 🤖 29. Confirme a execução com sucesso e validação de envio do relatório.

![Execução com Sucesso da Função Orquestradora](1%20\(29\).png)

- 🤖 30. Confira sua caixa de e-mail e valide o recebimento do Relatório Diário de Análise de Vendas com os itens pedidos.

![E-mail com Relatório Final Recebido](1%20\(28\).png)

---

## 👨‍💻 Expert

<p>
    <img 
      align=left 
      margin=10 
      width=80 
      src="https://avatars.githubusercontent.com/u/105826184?v=4"
    />
    <p>&nbsp&nbsp&nbspIsmael Medeiros<br>
    &nbsp&nbsp&nbsp
    <a 
        href="https://github.com/ism-dev-codes">
        GitHub
    </a>
    &nbsp;|&nbsp;
    <a 
        href="https://www.linkedin.com/in/ismael-medeiros">
        LinkedIn
    </a>
    &nbsp;|&nbsp;
    <a 
        href="https://www.instagram.com/ismaelsmedeiros?igsh=YXA1OW1mNXhkNmVy">
        Instagram
    </a>
    &nbsp;|&nbsp;</p>
</p>
<br/><br/>
<p>
