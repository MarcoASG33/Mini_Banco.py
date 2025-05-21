# RESETAR BANCO DE DADOS COMPLETAMENTE
def resetar_banco_de_dados():
    confirmacao = input("❗ Deseja realmente resetar o banco de dados e apagar todos os usuários? (s/n): ").strip().lower()
    if confirmacao == 's':
        if os.path.exists("usuarios.json"):
            os.remove("usuarios.json")
            print("✅ Banco de dados apagado com sucesso!")
        else:
            print("⚠️ Arquivo de banco de dados não encontrado.")

        for arquivo in os.listdir():
            if arquivo.endswith("_historico.txt"):
                os.remove(arquivo)
        print("✅ Todos os históricos de transações também foram apagados.")
        exit()

# PERGUNTA INICIAL PARA RESETAR
resetar = input("Deseja resetar o banco de dados e histórico de transações? (s/n): ").strip().lower()
if resetar == 's':
    resetar_banco_de_dados()
