# README.md sobre o repositório monitor-motores-python falando sobre como funciona o projeto

## Resumo
Ele é um sistema de monitoramento de motores industriais, atualmente na versão 1.0.0, que faz a leitura de várias informações deles (temperatura, vibração, rpm, corrente, potência kw, setor e id), válida e análisa elas e no final cálcula suas estatisticas com base nelas.

---

## Funcionamento

Ele recebe as informações de várias máquinas, válida essas informações para ver se estão corretas e depois faz uma análise delas checando os limites definidos, para ver como está o status da máquina, se está normal, se precisa de atenção ou está em alerta. Após a análise ele faz um cálculo das estatisticas médias e totais entre os motores para no fim mostrar um resumo de todas as informações adquiridas durante o funcionamento do sistema e também os alertas encontrados em cada máquina, também mostra horário e data de sua execução. 

A seguir será mostrado o código, com um breve resumo do que está em cada parte e no fim o que é exibido após sua execução.

---

## Código do sistema

### Trecho com as leituras e limites

```
from datetime import datetime

SISTEMA_NOME = "Monitoramento de Motores Industriais"
VERSAO = "1.0.0"

LIMITES = {
    "temperatura_max": 80.0,
    "corrente_max": 18.5,
    "vibracao_max": 4.5,
    "rpm_min": 1400,
    "rpm_max": 1800,
}

LEITURAS_MOTORES = [
    {"id": "MTR-01", "setor": "Estamparia", "temperatura": 68.4, "corrente": 14.2, "vibracao": 2.1, "rpm": 1750, "potencia_kw": 7.5},
    {"id": "MTR-02", "setor": "Usinagem", "temperatura": 84.7, "corrente": 19.3, "vibracao": 5.2, "rpm": 1690, "potencia_kw": 11.0},
    {"id": "MTR-03", "setor": "Bobinagem", "temperatura": 76.2, "corrente": 17.8, "vibracao": 4.7, "rpm": 1520, "potencia_kw": 5.5},
    {"id": "MTR-04", "setor": "Pintura", "temperatura": 61.0, "corrente": 11.5, "vibracao": 1.8, "rpm": 1780, "potencia_kw": 7.5},
]
```

### Trecho com a validação das informações da leitura

```
def validar_leitura(motor):
    campos_obrigatorios = ["id", "setor", "temperatura", "corrente", "vibracao", "rpm", "potencia_kw"]
    for campo in campos_obrigatorios:
        if campo not in motor or motor[campo] is None:
            return False
    if motor["temperatura"] < 0 or motor["corrente"] < 0 or motor["vibracao"] < 0 or motor["rpm"] < 0:
        return False
    return True


def obter_leituras_validas(lista_motores):
    return [m for m in lista_motores if validar_leitura(m)]
```

### Trecho com a análise dos status das máquinas e alertas

```
def analisar_alertas(motor):
    alertas = []
    if motor["temperatura"] > LIMITES["temperatura_max"]:
        alertas.append(f"Superaquecimento ({motor['temperatura']:.1f} C)")
    if motor["corrente"] > LIMITES["corrente_max"]:
        alertas.append(f"Sobrecorrente ({motor['corrente']:.1f} A)")
    if motor["vibracao"] > LIMITES["vibracao_max"]:
        alertas.append(f"Vibracao elevada ({motor['vibracao']:.1f} mm/s)")
    if motor["rpm"] < LIMITES["rpm_min"] or motor["rpm"] > LIMITES["rpm_max"]:
        alertas.append(f"Rotacao fora da faixa ({motor['rpm']} RPM)")

    if len(alertas) == 0:
        status = "NORMAL"
    elif len(alertas) == 1:
        status = "ATENCAO"
    else:
        status = "CRITICO"

    return status, alertas
```

### Trecho com o cálculo das estatisticas

```
def calcular_estatisticas(motores):
    total = len(motores)
    if total == 0:
        return {}

    temp_media = sum(m["temperatura"] for m in motores) / total
    corrente_media = sum(m["corrente"] for m in motores) / total
    potencia_total = sum(m["potencia_kw"] for m in motores)
    maior_temp = max(motores, key=lambda m: m["temperatura"])

    return {
        "total_motores": total,
        "temperatura_media": round(temp_media, 2),
        "corrente_media": round(corrente_media, 2),
        "potencia_total_kw": round(potencia_total, 2),
        "motor_mais_quente": maior_temp["id"],
    }
```

### Trecho com a exibição dos dados análisados e informações importantes

```
def exibir_painel(motores):
    print("=" * 64)
    print(f"{SISTEMA_NOME} - v{VERSAO}")
    print(f"Data/Hora da leitura: {datetime.now().strftime('%d/%m/%Y %H:%M:%S')}")
    print("=" * 64)

    for motor in motores:
        status, alertas = analisar_alertas(motor)
        print(
            f"[{status:<7}] {motor['id']} ({motor['setor']}) | "
            f"Temp: {motor['temperatura']} C | Corr: {motor['corrente']} A | "
            f"Vib: {motor['vibracao']} mm/s | RPM: {motor['rpm']}"
        )
        for alerta in alertas:
            print(f"  -> Alerta: {alerta}")

    stats = calcular_estatisticas(motores)
    print("-" * 64)
    print(
        f"Resumo: {stats['total_motores']} motores | "
        f"Temp. media: {stats['temperatura_media']} C | "
        f"Corr. media: {stats['corrente_media']} A | "
        f"Potencia total: {stats['potencia_total_kw']} kW | "
        f"Maior temp.: {stats['motor_mais_quente']}"
    )
    print("=" * 64)


if __name__ == "__main__":
    motores_validos = obter_leituras_validas(LEITURAS_MOTORES)
    exibir_painel(motores_validos)
```

## Resultados adquirido pelo sistema apartir das informações obtidas pela leitura

```
================================================================
Monitoramento de Motores Industriais - v1.0.0
Data/Hora da leitura: 26/11/2026 10:27:43
================================================================
[NORMAL ] MTR-01 (Estamparia) | Temp: 68.4 C | Corr: 14.2 A | Vib: 2.1 mm/s | RPM: 1750
[CRITICO] MTR-02 (Usinagem) | Temp: 84.7 C | Corr: 19.3 A | Vib: 5.2 mm/s | RPM: 1690
  -> Alerta: Superaquecimento (84.7 C)
  -> Alerta: Sobrecorrente (19.3 A)
  -> Alerta: Vibracao elevada (5.2 mm/s)
[ATENCAO] MTR-03 (Bobinagem) | Temp: 76.2 C | Corr: 17.8 A | Vib: 4.7 mm/s | RPM: 1520
  -> Alerta: Vibracao elevada (4.7 mm/s)
[NORMAL ] MTR-04 (Pintura) | Temp: 61.0 C | Corr: 11.5 A | Vib: 1.8 mm/s | RPM: 1780
----------------------------------------------------------------
Resumo: 4 motores | Temp. media: 72.58 C | Corr. media: 15.7 A | Potencia total: 31.5 kW | Maior temp.: MTR-02
================================================================
```

---

_README.md_ feito por **@mateus-welter**

Link no GitHub: [https://github.com/mateus-welter/monitor-motores-python/tree/main](https://github.com/mateus-welter/monitor-motores-python/tree/main)
