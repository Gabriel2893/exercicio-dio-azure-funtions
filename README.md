# exercicio-dio-azure-funtions
exercicio dio azure funtions


 Estrutura do Projeto
Certifique-se de ter o Azure Functions Core Tools instalado. Você pode criar uma nova aplicação do Azure Functions com o seguinte comando:

bash
Copiar código
func init cpf-validator --javascript
Entre na pasta do projeto:

bash
Copiar código
cd cpf-validator
Crie uma nova função HTTP:

bash
Copiar código
func new --template "HTTP trigger" --name ValidateCPF
2. Código da Função (index.js)
Substitua o conteúdo do arquivo ValidateCPF/index.js pelo seguinte:

javascript
Copiar código
const validateCPF = (cpf) => {
    cpf = cpf.replace(/\D/g, ''); // Remove caracteres não numéricos

    if (cpf.length !== 11 || /^(\d)\1+$/.test(cpf)) {
        return false;
    }

    const calcDigit = (factor, max) => {
        let total = 0;
        for (let i = 0; i < max; i++) {
            total += parseInt(cpf[i]) * (factor - i);
        }
        const remainder = total % 11;
        return remainder < 2 ? 0 : 11 - remainder;
    };

    const digit1 = calcDigit(10, 9);
    const digit2 = calcDigit(11, 10);

    return digit1 === parseInt(cpf[9]) && digit2 === parseInt(cpf[10]);
};

module.exports = async function (context, req) {
    const { cpf } = req.query;

    if (!cpf) {
        context.res = {
            status: 400,
            body: "Parâmetro 'cpf' é obrigatório.",
        };
        return;
    }

    const isValid = validateCPF(cpf);

    context.res = {
        status: 200,
        body: {
            cpf,
            isValid,
        },
    };
};
3. Configuração do function.json
O arquivo ValidateCPF/function.json já será gerado automaticamente com algo semelhante a isto:

json
Copiar código
{
    "bindings": [
        {
            "authLevel": "function",
            "type": "httpTrigger",
            "direction": "in",
            "name": "req",
            "methods": ["get", "post"]
        },
        {
            "type": "http",
            "direction": "out",
            "name": "res"
        }
    ]
}
4. Testando Localmente
Inicie o servidor localmente com:

bash
Copiar código
func start
A função estará disponível em http://localhost:7071/api/ValidateCPF. Você pode testá-la com uma requisição HTTP:

GET Request
bash
Copiar código
curl "http://localhost:7071/api/ValidateCPF?cpf=12345678909"
Exemplo de Resposta
json
Copiar código
{
    "cpf": "12345678909",
    "isValid": false
}
5. Implantação no Azure
Faça login no Azure CLI:

bash
Copiar código
az login
Crie um novo recurso do Azure Functions (se necessário) e publique a função:

bash
Copiar código
func azure functionapp publish <NOME_DA_SUA_APP>
Após a implantação, sua função estará disponível na URL fornecida pelo Azure, com o mesmo endpoint /api/ValidateCPF.

Essa abordagem utiliza o modelo de computação serverless da Azure Functions, garantindo escalabilidade e baixo custo.
