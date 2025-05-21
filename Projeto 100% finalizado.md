import json
import os
from datetime import datetime

# FUNÇÃO PARA FORMATAR VALORES EM REAIS
def formatar_moeda(valor):
    return f"R${valor:,.2f}".replace(",", "X").replace(".", ",").replace("X", ".")

# CARREGAR USUÁRIOS DO ARQUIVO JSON
def carregar_usuarios():
    if not os.path.exists("usuarios.json"):
        with open("usuarios.json", "w") as f:
            json.dump({}, f)
    with open("usuarios.json", "r") as f:
        return json.load(f)

# SALVAR USUÁRIOS NO JSON
def salvar_usuarios(usuarios):
    with open("usuarios.json", "w") as f:
        json.dump(usuarios, f, indent=4)

# REGISTRAR MOVIMENTAÇÕES NO HISTÓRICO
def registrar_transacao(username, tipo, valor):
    data = datetime.now().strftime("%d/%m/%Y %H:%M:%S")
    log = f"[{data}] {tipo} de {formatar_moeda(valor)}\n"
    with open(f"{username}_historico.txt", "a") as f:
        f.write(log)

# INÍCIO DO SISTEMA
print("=============================================== SEU BANCO ===============================================")

usuarios = carregar_usuarios()
tem_conta = input("\nVocê já tem conta? (s/n): ").strip().lower()

# LOGIN PARA QUEM JÁ TEM CADASTRO
if tem_conta == 's':
    username = input("Nome de usuário: ").strip()
    if username not in usuarios:
        print("\nUsuário não encontrado.")
        exit()

    senha = input("Senha: ").strip()
    if usuarios[username]['senha'] == senha:
        print(f"\nBem-vindo de volta, {username}!")
        total = usuarios[username]['saldo']
    else:
        recuperar = input("\nSenha incorreta. Deseja tentar recuperar sua senha? (s/n): ").strip().lower()
        if recuperar == 's':
            pergunta = usuarios[username].get('pergunta')
            resposta = usuarios[username].get('resposta')
            if pergunta and resposta:
                resposta_usuario = input(f"{pergunta} ").strip().lower()
                if resposta_usuario == resposta.lower():
                    nova_senha = input("Digite a nova senha: ").strip()
                    usuarios[username]['senha'] = nova_senha
                    salvar_usuarios(usuarios)
                    print("\nSenha redefinida com sucesso!")
                    total = usuarios[username]['saldo']
                else:
                    print("\nResposta incorreta. Encerrando o programa.")
                    exit()
            else:
                print("\nConta não tem pergunta secreta cadastrada. Encerrando.")
                exit()
        else:
            print("\nEncerrando o programa.")
            exit()

# NOVO CADASTRO
else:
    username = input("Crie seu nome de usuário: ").strip()
    if username in usuarios:
        print("\nEsse usuário já existe. Tente outro.")
        exit()
    senha = input("Crie sua senha: ").strip()
    pergunta = input("Cadastre uma pergunta secreta (ex: Qual o nome do seu primeiro pet?, "
    "faça essa pergunta para você mesmo a responde-la caso esqueça a senha:").strip()
    resposta = input("Resposta da pergunta secreta: ").strip()
    usuarios[username] = {"senha": senha, "saldo": 0, "pergunta": pergunta, "resposta": resposta}
    salvar_usuarios(usuarios)
    print("\nConta criada com sucesso!")
    total = 0

    deseja_depositar = input("Deseja realizar um depósito inicial? (s/n): ").strip().lower()
    if deseja_depositar == 's':
        print("\nVamos realizar o depósito com notas:")
        N_200 = int(input("Notas de R$200,00: "))
        N_100 = int(input("Notas de R$100,00: "))
        N_50 = int(input("Notas de R$50,00: "))
        N_20 = int(input("Notas de R$20,00: "))
        N_10 = int(input("Notas de R$10,00: "))
        N_5 = int(input("Notas de R$5,00: "))
        N_2 = int(input("Notas de R$2,00: "))

        total = (N_200 * 200 + N_100 * 100 + N_50 * 50 +
                 N_20 * 20 + N_10 * 10 + N_5 * 5 + N_2 * 2)
        usuarios[username]['saldo'] = total
        salvar_usuarios(usuarios)
        registrar_transacao(username, "Depósito inicial", total)
        print(f"\nDepósito realizado. Saldo atual: {formatar_moeda(total)}")

# MENU PRINCIPAL
while True:
    print("\n==================== MENU PRINCIPAL ====================")
    print("1 - Consultar saldo")
    print("2 - Saques")
    print("3 - Transferências")
    print("4 - Depósitos")
    print("5 - Ver histórico de transações")
    print("6 - Sair")
    print("========================================================")

    opcao = input("Escolha uma opção: ")

    if opcao == '1':
        print(f"\nSaldo atual: {formatar_moeda(total)}")

    elif opcao == '2':
        valor_saque = float(input(f"\nQual valor deseja sacar? Saldo: {formatar_moeda(total)}: "))
        if valor_saque <= total:
            total -= valor_saque
            print(f"\nSaque realizado no valor de {formatar_moeda(valor_saque)}! Novo saldo: {formatar_moeda(total)}")
            registrar_transacao(username, "Saque", valor_saque)
        else:
            print("\nSaldo insuficiente para saque.")

    elif opcao == '3':
        destinatario = input("\nDigite o nome do usuário para quem deseja transferir: ").strip()
        if destinatario == username:
            print("\nVocê não pode transferir para si mesmo.")
        elif destinatario not in usuarios:
            print("\nUsuário destinatário não encontrado.")
        else:
            valor = float(input("Digite o valor a ser transferido: "))
            if valor <= 0:
                print("\nValor inválido.")
            elif valor > total:
                print("\nSaldo insuficiente para a transferência.")
            else:
                total -= valor
                usuarios[destinatario]['saldo'] += valor
                print(f"\nTransferência de {formatar_moeda(valor)} realizada para {destinatario}.")
                print(f"Seu novo saldo: {formatar_moeda(total)}")
                registrar_transacao(username, f"Transferência para {destinatario}", valor)
                registrar_transacao(destinatario, f"Recebido de {username}", valor)

    elif opcao == '4':
        print("\nDepósito com notas:")
        N_200 = int(input("Notas de R$200,00: "))
        N_100 = int(input("Notas de R$100,00: "))
        N_50 = int(input("Notas de R$50,00: "))
        N_20 = int(input("Notas de R$20,00: "))
        N_10 = int(input("Notas de R$10,00: "))
        N_5 = int(input("Notas de R$5,00: "))
        N_2 = int(input("Notas de R$2,00: "))

        deposito = (N_200 * 200 + N_100 * 100 + N_50 * 50 +
                    N_20 * 20 + N_10 * 10 + N_5 * 5 + N_2 * 2)
        total += deposito
        print(f"\nDepósito de {formatar_moeda(deposito)} realizado. Novo saldo: {formatar_moeda(total)}")
        registrar_transacao(username, "Depósito", deposito)

    elif opcao == '5':
        historico_path = f"{username}_historico.txt"
        if os.path.exists(historico_path):
            print("\n===== HISTÓRICO DE TRANSAÇÕES =====")
            with open(historico_path, "r") as f:
                print(f.read())
        else:
            print("\nNenhuma transação registrada ainda.")

    elif opcao == '6':
        print(f"\nSaindo... Obrigado por utilizar nossos serviços, {username}!")
        break

    else:
        print("\nOpção inválida. Tente novamente.")

    usuarios[username]['saldo'] = total
    salvar_usuarios(usuarios)
    
