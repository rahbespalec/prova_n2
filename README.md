# prova_n2
1 - NA PAWSTA DO PROJETO, CRIAR UMA PASTA CHAMADA "Assets" e colocar o arquivo cvendas.csv lá
```
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;

namespace ProcessamentoVendas
{
    // Classes Venda, VendedorResumo e ProdutoQuantidade permanecem as mesmas
    public class Venda
    {
        public string Vendedor { get; set; }
        public string Produto { get; set; }
        public int Quantidade { get; set; }
        public decimal Valor { get; set; }
        public decimal ValorTotal => Quantidade * Valor;

        public override string ToString()
        {
            return $"Vendedor: {Vendedor}, Produto: {Produto}, Quantidade: {Quantidade}, " +
                   $"Valor Unitário: R${Valor:F2}, Valor Total: R${ValorTotal:F2}";
        }
    }

    public class ProcessadorVendas
    {
        private List<Venda> _vendas;
        private readonly string _caminhoArquivo;

        public ProcessadorVendas()
        {
            // Obtém o caminho do diretório do projeto
            string diretorioAtual = Directory.GetCurrentDirectory();
            string diretorioProjeto = Path.GetFullPath(Path.Combine(diretorioAtual, @"..\.."));
            _caminhoArquivo = Path.Combine(diretorioProjeto, "Assets", "vendas.csv");
            _vendas = new List<Venda>();

            Console.WriteLine($"Tentando ler arquivo em: {_caminhoArquivo}"); // Debug
        }

        public void LerArquivo()
        {
            try
            {
                // Verifica se a pasta Assets existe
                string pastaAssets = Path.GetDirectoryName(_caminhoArquivo);
                if (!Directory.Exists(pastaAssets))
                {
                    throw new DirectoryNotFoundException($"Pasta Assets não encontrada em: {pastaAssets}");
                }

                // Verifica se o arquivo existe
                if (!File.Exists(_caminhoArquivo))
                {
                    throw new FileNotFoundException($"Arquivo VENDAS.CSV não encontrado em: {_caminhoArquivo}");
                }

                string[] linhas = File.ReadAllLines(_caminhoArquivo);
                bool primeiraLinha = true;

                foreach (string linha in linhas)
                {
                    if (primeiraLinha)
                    {
                        primeiraLinha = false;
                        continue;
                    }

                    try
                    {
                        string[] dados = linha.Split(',');
                        if (dados.Length == 4)
                        {
                            _vendas.Add(new Venda
                            {
                                Vendedor = dados[0].Trim(),
                                Produto = dados[1].Trim(),
                                Quantidade = int.Parse(dados[2].Trim()),
                                Valor = decimal.Parse(dados[3].Trim())
                            });
                        }
                    }
                    catch (Exception ex)
                    {
                        Console.WriteLine($"Erro ao processar linha: {linha}");
                        Console.WriteLine($"Erro: {ex.Message}");
                    }
                }
            }
            catch (Exception ex)
            {
                throw new Exception($"Erro ao ler arquivo: {ex.Message}");
            }
        }

        public List<Venda> OrdenarPorValorTotal()
        {
            return _vendas.OrderByDescending(v => v.ValorTotal).ToList();
        }

        public List<Venda> BuscarVendasPorVendedor(string vendedor)
        {
            return _vendas.Where(v =>
                v.Vendedor.Equals(vendedor, StringComparison.OrdinalIgnoreCase))
                .ToList();
        }

        public VendedorResumo GerarResumoVendedor(string vendedor)
        {
            var vendasVendedor = BuscarVendasPorVendedor(vendedor);
            return new VendedorResumo
            {
                Vendedor = vendedor,
                TotalVendas = vendasVendedor.Count,
                ValorTotalVendas = vendasVendedor.Sum(v => v.ValorTotal),
                ProdutosMaisVendidos = vendasVendedor
                    .GroupBy(v => v.Produto)
                    .OrderByDescending(g => g.Sum(v => v.Quantidade))
                    .Take(3)
                    .Select(g => new ProdutoQuantidade
                    {
                        Produto = g.Key,
                        Quantidade = g.Sum(v => v.Quantidade)
                    })
                    .ToList()
            };
        }
    }

    public class VendedorResumo
    {
        public string Vendedor { get; set; }
        public int TotalVendas { get; set; }
        public decimal ValorTotalVendas { get; set; }
        public List<ProdutoQuantidade> ProdutosMaisVendidos { get; set; }

        public override string ToString()
        {
            var produtosStr = string.Join("\n", ProdutosMaisVendidos.Select(p =>
                $"  - {p.Produto}: {p.Quantidade} unidades"));

            return $"Resumo do Vendedor: {Vendedor}\n" +
                   $"Total de Vendas: {TotalVendas}\n" +
                   $"Valor Total: R${ValorTotalVendas:F2}\n" +
                   $"Produtos Mais Vendidos:\n{produtosStr}";
        }
    }

    public class ProdutoQuantidade
    {
        public string Produto { get; set; }
        public int Quantidade { get; set; }
    }

    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Iniciando processamento de vendas...\n");
            try
            {
                var processador = new ProcessadorVendas();
                processador.LerArquivo();

                while (true)
                {
                    Console.WriteLine("\nEscolha uma opção:");
                    Console.WriteLine("1 - Ver todas as vendas ordenadas por valor total");
                    Console.WriteLine("2 - Buscar vendas por vendedor");
                    Console.WriteLine("3 - Sair");
                    Console.Write("\nOpção: ");

                    string opcao = Console.ReadLine();

                    switch (opcao)
                    {
                        case "1":
                            Console.WriteLine("\n=== Vendas ordenadas por valor total ===");
                            var vendasOrdenadas = processador.OrdenarPorValorTotal();
                            foreach (var venda in vendasOrdenadas)
                            {
                                Console.WriteLine(venda);
                            }
                            break;

                        case "2":
                            Console.WriteLine("\n=== Buscar vendas por vendedor ===");
                            Console.Write("Digite o nome do vendedor: ");
                            string vendedor = Console.ReadLine();

                            var vendasVendedor = processador.BuscarVendasPorVendedor(vendedor);
                            if (vendasVendedor.Any())
                            {
                                Console.WriteLine($"\nVendas de {vendedor}:");
                                foreach (var venda in vendasVendedor)
                                {
                                    Console.WriteLine(venda);
                                }

                                var resumo = processador.GerarResumoVendedor(vendedor);
                                Console.WriteLine("\n=== Resumo do Vendedor ===");
                                Console.WriteLine(resumo);
                            }
                            else
                            {
                                Console.WriteLine($"Nenhuma venda encontrada para o vendedor {vendedor}");
                            }
                            break;

                        case "3":
                            return;

                        default:
                            Console.WriteLine("Opção inválida!");
                            break;
                    }
                }
            }
            catch (Exception ex)
            {
                Console.WriteLine($"Erro: {ex.Message}");
            }
            finally
            {
                Console.WriteLine("\nPressione qualquer tecla para sair...");
                Console.ReadKey();
            }
        }
    }
}
```
