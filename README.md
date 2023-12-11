# S.O.L.I.D 

Os princípios S.O.L.I.D são foram criados a partir da análise e observação da orientação a objetos e design de projetos. Esses princípios visam a criação de um código com o mínimo de acoplamento possível, que são facilmente refatoráveis e tem sua leitura facilitada pela aplicação destes.

*SOLID são cinco princípios da programação orientada a objetos que facilitam no desenvolvimento de softwares, tornando-os fáceis de manter e estender. Esses princípios podem ser aplicados a qualquer linguagem de POO.*

Estes arquivo visa apresentar quais são estes conceitos, assim como, demonstrar exemplos 

## 1. SRP — Single Responsibility Principle (Princípio da Responsabilidade Única):

Este princípio define que as responsabilidades devem ser separadas conforme seu ator e devem definir apenas um objetivo a ser completo, ou seja, uma classe ou método deve atuar somente sobre uma única responsabilidade.

A violação do Single Responsibility Principle pode gerar alguns problemas, sendo eles:

- Falta de coesão — uma classe não deve assumir responsabilidades que não são suas;
- Alto acoplamento — Mais responsabilidades geram um maior nível de dependências, deixando o sistema engessado e frágil para alterações;
- Dificuldades na implementação de testes automatizados — É difícil de “mockar” esse tipo de classe;
- Dificuldades para reaproveitar o código;

### Exemplo de Aplicação do Princípio

Nosso primeiro contexto de exemplo é um cliente realizando login em uma plataforma. Como podemos ver abaixo no primeiro exemplo, além do efetuar o log in o método também faz a validação dos dados vindos do modelos de dados de log in, ferindo o princípio de responsabilidade única.

``` C#
namespace Modelos
{
    public class LogOnModel
    {
        public string username { get; set; }
        public string password { get; set; }
    }
}

namespace Controladores
{
    public class Login
    {
        #region Construtores
        //...
        #endregion

        #region Propriedades
        //...
        #endregion

        #region Métodos
        public void EfetuarLogIn(Modelos.LogOnModel model)
        {
            if (string.IsNullOrEmpty(username) || string.IsNullOrEmpty(password))
                throw new Exception("Favor preencher os dados de nome de usuário e senha para efetuar o log in.");

            if (username.Length <= 3)
                throw new Exception("O tamamnho do nome de usuário deve ser maior que 3 caracteres");

            if (password.Length <= 8)
                throw new Exception("O tamamnho da senha deve ser maior que 8 caracteres");

            //....
            // Seguir com login e criação de sessão para o usuário
        }
        #endregion
    }
}
```

Isso pode ser ajustado separando as validações em uma outra classe responsável apenas por validações em modelos de dados, dessa maneira, caso novas alterações sejam necessárias o código pode ser extendido sem problemas.

Essa separação também providencia uma leitura melhor do método principal e coesão no funcionamento das classes e métodos envolvidos.

``` C#
namespace Modelos
{
    public class LogOnModel
    {
        public string username { get; set; }
        public string password { get; set; }
    }
}

namespace Controladores
{
    public class Login
    {       
        #region Métodos
        public void EfetuarLogIn(Modelos.LogOnModel model)
        {
            Validacoes.Validador.ValidarPadraoDeDadosUsuario(model.username, model.password);

            // Seguir com login e criação de sessão para o usuário
        }

        //..
        #endregion
    }
}

namespace Validacoes
{
    public class Validador
    {
        public static void ValidarPadraoDeDadosUsuario(string username, string password)
        {
            if (string.IsNullOrEmpty(username) || string.IsNullOrEmpty(password))
                throw new Exception("Favor preencher os dados de nome de usuário e senha para efetuar o log in.");

            if (username.Length <= 3)
                throw new Exception("O tamamnho do nome de usuário deve ser maior que 3 caracteres");

            if (password.Length <= 8)
                throw new Exception("O tamamnho da senha deve ser maior que 8 caracteres");

            //....
        }
    }
}
```

## 2. OCP — Open-Closed Principle (Princípio Aberto-Fechado):

Esse princípio destaca que uma classe deve se manter com inalterada em relação ao seu escopo inicial mas deve possibilitar uma extensão fácil caso necessário.

De forma resumida podemos dizer que os objetos devem estar abertos para extensão, mas fechados para modificação.

### Exemplo de Aplicação do Princípio

Nosso contexto de exemplo é o carregamento de permissões em uma aplicação para um usuário. A primeiro momento separamos ambas as funções e validamos qual o tipo de usuário foi instanciando para verificar se o mesmo tem a pemissão retornar esta informação para o método pai.

A princípio podemos não ver problemas com esta implementação, entretanto, para extensões futuras, alterar uma classe já existente para adicionar um novo comportamento, faz com que o desenvolvedor corra um sério risco de introduzir bugs em algo que já estava funcionando.

``` C#
namespace Objetos {
    public class Usario
    {
        //..
    }

    public class UsarioAdmin
    {
        //..
    }

    public class Visitante
    {
        //..
    }    
}

namespace Controladores
{
    public class GerenciadorDePermissoes 
    {
        public bool PodeVisualizarTelaDeManutencao(object usuario)
        {
            if (usuario is Objetos.Usario)
                return false;

            if (usuario is Objetos.Visitante)
                return true;

            if (usuario is Objetos.UsarioAdmin)
                return true;

            return false;
        }
    }
}
```

Neste caso, para efetuar um ajuste seguindo o princípio, apenas separamos de forma que cada classe do tipo Usuario utilize a interface de usuário e através desta definimos atributos respectivos aquele tipo de usuário.

Ainda podemos fazer de forma que todos usuários padrões do sistema tenha sua permissão verificado em caso de alguma configuração especial.

``` C#
using System;

namespace Objetos
{
    public interface Usuario
    {
        int Id { get; set; }       
        bool PodeVisualizarTelaManutencao { get; set; }
    }
    public class UsuarioAdmin : Usuario
    {
        public int Id { get; set; }
        public UsuarioAdmin(int Id)
        {
            this.Id = Id;
        }  
        public bool PodeVisualizarTelaManutencao
        {
            get
            {
                return true;
            }
            set => throw new NotImplementedException();
        }       
    }

    public class UsuarioPadrao : Usuario
    {
        public int Id { get; set; }
        public UsuarioPadrao(int Id)
        {
            this.Id = Id;
        }        
        public bool PodeVisualizarTelaManutencao
        {
            get
            {
                return Controladores.GerenciadorDePermissoes.UsuarioTemPermissaoEspecial(this.Id);
            }
            set => throw new NotImplementedException();
        }
    }

    public class Visitante : Usuario
    {
        public int Id { get; set; }
        public bool PodeVisualizarTelaManutencao
        {
            get
            {
                return false;
            }
            set => throw new NotImplementedException();
        }
    }
}

namespace Controladores
{
    public class GerenciadorDePermissoes
    {
        public static bool UsuarioTemPermissaoEspecial(int Id)
        {
            //...
            return true;
        }
    }
}
```

## 3. LSP— Liskov Substitution Principle (Princípio da substituição de Liskov):

Para entendimento vamos vamos verificar a definição presente na Wikipédia: 

***Se S é um subtipo de T, então os objetos do tipo T, em um programa, podem ser substituídos pelos objetos de tipo S sem que seja necessário alterar as propriedades deste programa***

Exemplos de violação do LSP:

- Sobrescrever/implementar um método que não faz nada;
- Retornar valores de tipos diferentes da classe base;
- Lançar uma exceção inesperada;

### Exemplo de Não Aplicação do Princípio

``` C#
namespace Sobrescrição
{
    interface Carro
    {
        void DarPartida();
    }

    public class CarroEletrico : Carro
    {
        public void DarPartida()
        {
            //...
        }
    }
}


namespace ExceçãoSemDescrição
{
    interface Carro
    {
        void DarPartida();
    }

    public class CarroEletrico : Carro
    {
        public void DarPartida()
        {
            throw new System.NotImplementedException();
        }
    }
}


namespace ValoresDeTiposDiferentes
{
    interface Carro
    {
        void DarPartida();
    }

    public class CarroEletrico : Carro
    {
        public void DarPartida()
        {
            // No C# isso gera o Compiler Error CS0127
            if (this is CarroEletrico)
                return true;

            throw new System.NotImplementedException();
        }
    }
}

```

## 4. ISP — Interface Segregation Principle (Princípio da Segregação da Interface):

Este princípio define que ter diversas interfaces é melhor do que definir uma interface geral, ou seja, o princípio visa dar preferencia para a criação de interfaces mais específicas ao invés de interfaces genéricas.


### Exemplo de Não Aplicação do Princípio

``` C#
interface Veiculo
{
    double GasolinaNoTanque();
}

public class Caminhao : Veiculo
{
    public double GasolinaNoTanque()
    {
        throw new System.NotImplementedException();
    }
}

public class CarroEletrico : Veiculo
{
    // Força com que a classe de carros elétricos implemente o método
    // e fere o princípio.
    public double GasolinaNoTanque()
    {
        throw new System.NotImplementedException();
    }
}
```

Para corrigir especificamos ainda mais as interfaces a serem utilizadas para abstrair as especificidades.

``` C#
interface Veiculo
{
    string Marca { get; set; }
    string Modelo { get; set; }
}

interface VeiculoFlex : Veiculo
{
    double GasolinaNoTanque();
}

interface VeiculoEletrico : Veiculo
{
    double PorcentagemDeBateria();
}

public class Caminhao : VeiculoFlex
{
    string Veiculo.Marca { get => throw new System.NotImplementedException(); set => throw new System.NotImplementedException(); }
    string Veiculo.Modelo { get => throw new System.NotImplementedException(); set => throw new System.NotImplementedException(); }

    public double GasolinaNoTanque()
    {
        throw new System.NotImplementedException();
    } 
}

public class CarroEletrico : VeiculoEletrico
{
    public string Marca { get => throw new System.NotImplementedException(); set => throw new System.NotImplementedException(); }
    public string Modelo { get => throw new System.NotImplementedException(); set => throw new System.NotImplementedException(); }  

    public double PorcentagemDeBateria()
    {
        throw new System.NotImplementedException();
    }
}
```


## 5. DIP — Dependency Inversion Principle (Princípio da Inversão de Dependência):

De acordo com Uncle Bob, esse princípio pode ser definido da seguinte forma:

1. Módulos de alto nível não devem depender de módulos de baixo nível. Ambos devem depender da abstração.
2. Abstrações não devem depender de detalhes. Detalhes devem depender de abstrações.

Importante: Inversão de Dependência não é igual a Injeção de Dependência, fique ciente disso! A Inversão de Dependência é um princípio (Conceito) e a Injeção de Dependência é um padrão de projeto (Design Pattern).

### Exemplo de Não Aplicação do Princípio

*Nesse trecho de código temos um alto nível de acoplamento, isso ocorre pois a classe tem a responsabilidade de criar uma instância da classe SqlConnection! Para reaproveitar essa classe em outro sistema, teriamos obrigatoriamente de levar a classe SqlConnection junto, portanto, temos um forte acoplamento aqui.*

``` C#
using System.Data.SqlClient;

public class RepositorioUsuarios
{
    private SqlConnection connection { get; set; }

    public RepositorioUsuarios()
    {
        this.connection = new SqlConnection();
    }
}
```

Para resolver esse problema de acoplamento, podemos refatorar nosso código da seguinte forma.

``` C#
using System.Data.SqlClient;

public class RepositorioUsuarios
{
    private SqlConnection connection { get; set; }

    public RepositorioUsuarios(SqlConnection ct)
    {
        this.connection = ct;
    }
}
```
