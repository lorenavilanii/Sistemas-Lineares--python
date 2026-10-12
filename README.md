# Sistemas-Lineares--python
import tkinter as tk
from tkinter import messagebox, scrolledtext

class ResolvedorSistemaGUI:
    def __init__(self, root):
        self.root = root
        self.root.title("Resolvedor de Sistemas Lineares (Escalonamento)")
        self.root.geometry("800x700")
        self.root.config(bg="#f4f4f9")

        self.letras_variaveis = ["X", "Y", "Z", "W", "A", "B", "C", "D", "E", "F"]
        self.entries = []

        self.criar_widgets()

    def criar_widgets(self):
        titulo = tk.Label(
            self.root, text="Resolução de Sistemas Lineares (Até 10x10)", 
            font=("Arial", 16, "bold"), bg="#f4f4f9", fg="#333333"
        )
        titulo.pack(pady=15)

        # Frame Superior: Configuração e Limpeza de Campos
        frame_config = tk.Frame(self.root, bg="#f4f4f9")
        frame_config.pack(pady=5)

        lbl_tamanho = tk.Label(frame_config, text="Tamanho (N x N, max 10):", font=("Arial", 11), bg="#f4f4f9")
        lbl_tamanho.pack(side=tk.LEFT, padx=5)

        self.entry_n = tk.Entry(frame_config, font=("Arial", 11), width=5, justify="center")
        self.entry_n.pack(side=tk.LEFT, padx=5)
        self.entry_n.insert(0, "3")

        btn_gerar = tk.Button(
            frame_config, text="Gerar Campos", font=("Arial", 10, "bold"), 
            bg="#007acc", fg="white", padx=10, pady=2, command=self.gerar_campos_matriz
        )
        btn_gerar.pack(side=tk.LEFT, padx=5)

        btn_limpar_campos = tk.Button(
            frame_config, text="Limpar Campos", font=("Arial", 10, "bold"), 
            bg="#d9534f", fg="white", padx=10, pady=2, command=self.limpar_campos
        )
        btn_limpar_campos.pack(side=tk.LEFT, padx=5)

        # Frame Central: Matriz com Scrollbar
        self.frame_matriz_container = tk.Frame(self.root, bg="#ffffff", bd=2, relief="groove")
        self.frame_matriz_container.pack(pady=10, fill="both", expand=True, padx=20)

        self.canvas = tk.Canvas(self.frame_matriz_container, bg="#ffffff", highlightthickness=0)
        self.scrollbar = tk.Scrollbar(self.frame_matriz_container, orient="vertical", command=self.canvas.yview)
        self.scroll_frame = tk.Frame(self.canvas, bg="#ffffff")

        self.scroll_frame.bind(
            "<Configure>",
            lambda e: self.canvas.configure(scrollregion=self.canvas.bbox("all"))
        )
        self.canvas.create_window((0, 0), window=self.scroll_frame, anchor="nw")
        self.canvas.configure(yscrollcommand=self.scrollbar.set)

        self.canvas.pack(side="left", fill="both", expand=True)
        self.scrollbar.pack(side="right", fill="y")

        # Frame Inferior: Botões de Ação (Resolver e Limpar Resultado)
        frame_inferior = tk.Frame(self.root, bg="#f4f4f9")
        frame_inferior.pack(pady=10, fill="x", padx=20)

        frame_botoes = tk.Frame(frame_inferior, bg="#f4f4f9")
        frame_botoes.pack(fill="x", pady=5)

        btn_resolver = tk.Button(
            frame_botoes, text="Resolver Sistema", font=("Arial", 11, "bold"), 
            bg="#28a745", fg="white", pady=5, command=self.resolver_sistema
        )
        btn_resolver.pack(side=tk.LEFT, fill="x", expand=True, padx=(0, 5))

        btn_limpar_resultado = tk.Button(
            frame_botoes, text="Limpar Resultado", font=("Arial", 11, "bold"), 
            bg="#6c757d", fg="white", pady=5, command=self.limpar_resultado
        )
        btn_limpar_resultado.pack(side=tk.RIGHT, fill="x", expand=True, padx=(5, 0))

        # Caixa de Texto para Resultados
        self.txt_resultado = scrolledtext.ScrolledText(
            self.root, height=10, font=("Consolas", 10), bg="#1e1e1e", fg="#d4d4d4"
        )
        self.txt_resultado.pack(padx=20, pady=10, fill="both")

        self.gerar_campos_matriz()

    def formatar_numero(self, val):
        """Remove casas decimais se o número for inteiro, caso contrário exibe com decimais."""
        if abs(val - round(val)) < 1e-9:
            return str(int(round(val)))
        else:
            return f"{val:.2f}"

    def gerar_campos_matriz(self):
        try:
            n = int(self.entry_n.get())
            if n < 1 or n > 10:
                messagebox.showerror("Erro", "O tamanho do sistema deve estar entre 1 e 10.")
                return
        except ValueError:
            messagebox.showerror("Erro", "Digite um número inteiro válido para o tamanho.")
            return

        for widget in self.scroll_frame.winfo_children():
            widget.destroy()
        self.entries = []

        letras_usadas = self.letras_variaveis[:n]
        vars_str = ", ".join(letras_usadas)

        lbl_instrucao = tk.Label(
            self.scroll_frame, text=f"Insira os coeficientes ({vars_str}) e o termo independente (b):", 
            font=("Arial", 10, "italic"), bg="#ffffff", fg="#555555"
        )
        lbl_instrucao.grid(row=0, column=0, columnspan=(n * 2 + 3), sticky="w", padx=5, pady=5)

        for i in range(n):
            linha_entries = []
            for j in range(n):
                e = tk.Entry(self.scroll_frame, font=("Arial", 10), width=6, justify="center")
                e.grid(row=i+1, column=j*2, padx=2, pady=3)
                e.insert(0, "0")
                linha_entries.append(e)

                sinal_str = " + " if j < n-1 else " = "
                lbl_x = tk.Label(self.scroll_frame, text=f"{self.letras_variaveis[j]}" + sinal_str, bg="#ffffff", font=("Arial", 9, "bold"))
                lbl_x.grid(row=i+1, column=j*2+1, padx=1)

            e_termo = tk.Entry(self.scroll_frame, font=("Arial", 10), width=6, justify="center", fg="blue")
            e_termo.grid(row=i+1, column=n*2+2, padx=5, pady=3)
            e_termo.insert(0, "0")
            linha_entries.append(e_termo)

            self.entries.append(linha_entries)

    def limpar_campos(self):
        """Esvazia todas as caixas de texto de entrada da matriz."""
        for linha in self.entries:
            for entry in linha:
                entry.delete(0, tk.END)

    def limpar_resultado(self):
        """Limpa a caixa de texto do painel de resultados."""
        self.txt_resultado.delete("1.0", tk.END)

    def formatar_sistema_texto(self, matriz):
        n = len(matriz)
        linhas_str = []
        for i in range(n):
            linha_str = []
            for j in range(n):
                coef = matriz[i][j]
                
                if j == 0:
                    sinal = "-" if coef < 0 else ""
                else:
                    sinal = " - " if coef < 0 else " + "
                
                val_str = self.formatar_numero(abs(coef))
                letra = self.letras_variaveis[j]
                linha_str.append(f"{sinal}{val_str}*{letra}")
            
            expressao = "".join(linha_str)
            if expressao.startswith(" + "):
                expressao = expressao[3:]
            elif expressao.startswith(" +"):
                expressao = expressao[2:]
                
            termo_indep = self.formatar_numero(matriz[i][n])
            linhas_str.append(f"  {expressao} = {termo_indep}")
        return "\n".join(linhas_str)

    def resolver_sistema(self):
        try:
            n = len(self.entries)
            matriz = [[0.0 for _ in range(n + 1)] for _ in range(n)]

            for i in range(n):
                for j in range(n + 1):
                    val_str = self.entries[i][j].get().strip()
                    matriz[i][j] = float(val_str) if val_str != "" else 0.0

            matriz_original = [linha[:] for linha in matriz]
            matriz_trabalho = [linha[:] for linha in matriz]

            self.txt_resultado.delete("1.0", tk.END)

            self.txt_resultado.insert(tk.END, "=== SISTEMA ORIGINAL ===\n")
            self.txt_resultado.insert(tk.END, self.formatar_sistema_texto(matriz_original) + "\n\n")

            # Escalonamento por Multiplicação Cruzada (Mantém números inteiros)
            for i in range(n):
                max_linha = i
                for k in range(i + 1, n):
                    if abs(matriz_trabalho[k][i]) > abs(matriz_trabalho[max_linha][i]):
                        max_linha = k
                
                if max_linha != i:
                    matriz_trabalho[i], matriz_trabalho[max_linha] = matriz_trabalho[max_linha], matriz_trabalho[i]
                
                if abs(matriz_trabalho[i][i]) < 1e-12:
                    continue
                
                # Multiplica cruzado para zerar abaixo do pivô sem gerar decimais
                for k in range(i + 1, n):
                    pivo = matriz_trabalho[i][i]
                    coef = matriz_trabalho[k][i]
                    if abs(coef) < 1e-12:
                        continue
                    for j in range(i, n + 1):
                        matriz_trabalho[k][j] = matriz_trabalho[k][j] * pivo - matriz_trabalho[i][j] * coef

            self.txt_resultado.insert(tk.END, "=== SISTEMA ESCALONADO ===\n")
            self.txt_resultado.insert(tk.END, self.formatar_sistema_texto(matriz_trabalho) + "\n\n")

            solucao = [0.0] * n
            status = "SOLUCAO_UNICA"

            for i in range(n):
                soma_coef = sum(abs(matriz_trabalho[i][j]) for j in range(n))
                if soma_coef < 1e-12 and abs(matriz_trabalho[i][n]) > 1e-12:
                    status = "SEM_SOLUCAO"
                    break
                elif soma_coef < 1e-12 and abs(matriz_trabalho[i][n]) < 1e-12:
                    status = "INFINITAS"

            self.txt_resultado.insert(tk.END, "=== SOLUÇÃO ===\n")
            if status == "SEM_SOLUCAO":
                self.txt_resultado.insert(tk.END, "  O sistema não possui solução (Sistema Impossível).\n")
            elif status == "INFINITAS":
                self.txt_resultado.insert(tk.END, "  O sistema possui infinitas soluções (Sistema Indeterminado).\n")
            else:
                for i in range(n - 1, -1, -1):
                    if abs(matriz_trabalho[i][i]) < 1e-12:
                        status = "INFINITAS"
                        break
                    soma = sum(matriz_trabalho[i][j] * solucao[j] for j in range(i + 1, n))
                    solucao[i] = (matriz_trabalho[i][n] - soma) / matriz_trabalho[i][i]
                
                if status == "INFINITAS":
                    self.txt_resultado.insert(tk.END, "  O sistema possui infinitas soluções (Sistema Indeterminado).\n")
                else:
                    for i, val in enumerate(solucao):
                        letra = self.letras_variaveis[i]
                        val_formatado = self.formatar_numero(val)
                        self.txt_resultado.insert(tk.END, f"  {letra} = {val_formatado}\n")

        except ValueError:
            messagebox.showerror("Erro de Formato", "Por favor, preencha os campos apenas com números válidos.")

if __name__ == "__main__":
    root = tk.Tk()
    app = ResolvedorSistemaGUI(root)
    root.mainloop()
