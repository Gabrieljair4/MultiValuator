# MultiValuator
Script inteligente para Valuation que automatiza o cálculo do preço-teto. Através da integração com dados de mercado, o software aplica simultaneamente os métodos de Graham, Bazin, Gordon e DCF. O sistema entrega uma análise comparativa e uma média ponderada precisa, ideal para investidores que buscam segurança na margem de segurança.
import yfinance as ticker_data

def calcular_preco_teto(ticker_symbol):
    # 1. Coleta de dados via yfinance
    acao = ticker_data.Ticker(ticker_symbol)
    info = acao.info
    
    # Variáveis necessárias (com valores padrão caso falte algum dado)
    lpa = info.get('trailingEps', 0) # Lucro por Ação
    vpa = info.get('bookValue', 0)   # Valor Patrimonial por Ação
    dy_pago = info.get('dividendRate', 0) # Dividendos pagos no último ano
    crescimento_esperado = 0.05 # Suposição de 5% de crescimento para Gordon/DCF
    taxa_desconto = 0.10        # Taxa de desconto de 10% (WACC estimado)
    
    # --- MÉTODO 1: GRAHAM ---
    # Fórmula: Raiz Quadrada de (22.5 * LPA * VPA)
    if lpa > 0 and vpa > 0:
        preco_graham = (22.5 * lpa * vpa) ** 0.5
    else:
        preco_graham = 0

    # --- MÉTODO 2: BAZIN ---
    # Fórmula: Dividendos / 0.06 (Expectativa de 6% de rendimento)
    preco_bazin = dy_pago / 0.06

    # --- MÉTODO 3: GORDON (Modelo de Crescimento de Dividendos) ---
    # Fórmula: D1 / (k - g) -> D1 = Próximo dividendo, k = custo capital, g = crescimento
    if taxa_desconto > crescimento_esperado:
        preco_gordon = (dy_pago * (1 + crescimento_esperado)) / (taxa_desconto - crescimento_esperado)
    else:
        preco_gordon = 0

    # --- MÉTODO 4: DCF SIMPLIFICADO (Fluxo de Caixa Descontado) ---
    # Projeção simples de 5 anos baseada no Fluxo de Caixa Livre
    fcf = info.get('freeCashflow', 0) / info.get('sharesOutstanding', 1)
    preco_dcf = 0
    for i in range(1, 6):
        preco_dcf += (fcf * (1 + crescimento_esperado)**i) / (1 + taxa_desconto)**i

    # --- MÉDIA PONDERADA ---
    # Pesos: Graham (20%), Bazin (30%), Gordon (25%), DCF (25%)
    media_ponderada = (preco_graham * 0.20) + (preco_bazin * 0.30) + \
                      (preco_gordon * 0.25) + (preco_dcf * 0.25)

    return {
        "Graham": preco_graham,
        "Bazin": preco_bazin,
        "Gordon": preco_gordon,
        "DCF": preco_dcf,
        "Média Final": media_ponderada
    }

# Execução do Programa
ticker = input("Digite o ticker da ação (ex: PETR4.SA): ").upper()
resultados = calcular_preco_teto(ticker)

print(f"\n--- Análise de Preço-Teto para {ticker} ---")
for metodo, valor in resultados.items():
    print(f"{metodo}: R$ {valor:.2f}")
