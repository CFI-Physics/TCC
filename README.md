import math
import matplotlib.pyplot as plt
from mpl_toolkits.mplot3d import Axes3D
import numpy as np

def calcular_ds2(evento):
    t, x, y = evento
    return -(t ** 2) + (x ** 2 + y ** 2)

def classificar_evento(ds2):
    regiao = ""
    if ds2 < 0:
        regiao = "tipo tempo"
    elif ds2 == 0:
        regiao = "tipo luz"
    else:
        regiao = "tipo espaço"
    return regiao

def determina_regiao(evento):
    t, x, y = evento
    delta_t = t
    delta_x = math.sqrt(x**2 + y**2)

    regiao = ""
    if delta_t > 0 and delta_t > delta_x:
        regiao = "Futuro"
    elif delta_t < 0 and delta_t < delta_x:
        regiao = "Passado"
    elif delta_t < delta_x:
        regiao = "Em outro lugar"
    else:  # delta_t == delta_x
        regiao = "Superfície do Cone"
    return regiao

def processar_eventos(arquivo):
    eventos = []
    with open(arquivo, 'r') as file:
        for linha in file:
            valores = linha.strip().split(',')
            if len(valores) != 3:
                print(f"Linha inválida (esperado 3 valores, obtido {len(valores)}): {linha.strip()}")
                continue
            try:
                evento = tuple(map(float, valores))
                eventos.append(evento)
            except ValueError:
                print(f"Erro de conversão para float na linha: {linha.strip()}")

    return [(evento, calcular_ds2(evento), classificar_evento(calcular_ds2(evento)), determina_regiao(evento)) for evento in eventos]

def plot_lightcone_3d(eventos):
    fig = plt.figure()
    ax = fig.add_subplot(111, projection='3d')

    # Criar o cone de luz positivo
    theta = np.linspace(0, 2*np.pi, 100)
    z_pos = np.linspace(0, 10, 100)
    t, z_pos = np.meshgrid(theta, z_pos)
    x_pos = z_pos * np.cos(t)
    y_pos = z_pos * np.sin(t)
    ax.plot_surface(x_pos, y_pos, z_pos, color='b', alpha=0.5)

    # Criar o cone de luz negativo
    z_neg = np.linspace(0, -10, 100)
    t, z_neg = np.meshgrid(theta, z_neg)
    x_neg = z_neg * np.cos(t)
    y_neg = z_neg * np.sin(t)
    ax.plot_surface(x_neg, y_neg, z_neg, color='b', alpha=0.5)

    # Plotar os eventos
    for i, (evento, _, _, _) in enumerate(eventos):
        t, x, y = evento
        ax.scatter(x, y, t, c='r' if t > 0 else 'b')
        ax.text(x, y, t, f"E{i+1}", size=10, zorder=1, ha='center')

    # Configurar rótulos dos eixos e título
    ax.set_xlabel('x')
    ax.set_ylabel('y')
    ax.set_zlabel('t')
    ax.set_title('Light Cone 3D')

    plt.show()

def main():
    arquivo = 'evento2.txt'
    classificacoes = processar_eventos(arquivo)
    for evento, ds2, tipo_evento, cone_classificacao in classificacoes:
        print(f"Evento: {evento}, ds²: {ds2}, Classificação: {tipo_evento}, Cone de Luz: {cone_classificacao}")

    plot_lightcone_3d(classificacoes)

if __name__ == "__main__":
    main()
