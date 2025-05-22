import 'package:flutter/material.dart';

void main() {
  runApp(const ForcaAtivaApp());
}

class ForcaAtivaApp extends StatelessWidget {
  const ForcaAtivaApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Força Ativa',
      theme: ThemeData(
        primarySwatch: Colors.deepPurple,
        colorScheme: ColorScheme.fromSwatch().copyWith(secondary: Colors.green),
      ),
      debugShowCheckedModeBanner: false,
      home: const HomePage(),
    );
  }
}

class Aluno {
  String nome;
  int frequenciaSemanal;
  int pagamentosAtrasados;
  int participacaoAtividades;

  Aluno({
    required this.nome,
    required this.frequenciaSemanal,
    required this.pagamentosAtrasados,
    required this.participacaoAtividades,
  });

  bool get riscoAlto {
    final mes = DateTime.now().month;
    final ehInverno = mes == 6 || mes == 7 || mes == 8;
    return frequenciaSemanal < 2 || pagamentosAtrasados > 1 || participacaoAtividades < 50 || ehInverno;
  }
}

class HomePage extends StatefulWidget {
  const HomePage({super.key});
  @override
  State<HomePage> createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  final List<Aluno> alunos = [];

  final nomeController = TextEditingController();
  final frequenciaController = TextEditingController();
  final atrasoController = TextEditingController();
  final participacaoController = TextEditingController();

  int paginaAtual = 0;

  void adicionarAluno() {
    final nome = nomeController.text.trim();
    final freq = int.tryParse(frequenciaController.text) ?? -1;
    final atrasos = int.tryParse(atrasoController.text) ?? -1;
    final participacao = int.tryParse(participacaoController.text) ?? -1;

    if (nome.isEmpty || freq < 0 || atrasos < 0 || participacao < 0) {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text('Preencha todos os campos corretamente!')),
      );
      return;
    }

    setState(() {
      alunos.add(Aluno(
        nome: nome,
        frequenciaSemanal: freq,
        pagamentosAtrasados: atrasos,
        participacaoAtividades: participacao,
      ));
      nomeController.clear();
      frequenciaController.clear();
      atrasoController.clear();
      participacaoController.clear();
      paginaAtual = 1; // Vai pra lista de alunos
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Força Ativa'),
        backgroundColor: Colors.deepPurple,
      ),
      body: paginaAtual == 0 ? buildCadastro() : buildLista(),
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: paginaAtual,
        onTap: (index) => setState(() => paginaAtual = index),
        items: const [
          BottomNavigationBarItem(icon: Icon(Icons.person_add), label: 'Cadastrar'),
          BottomNavigationBarItem(icon: Icon(Icons.list), label: 'Alunos'),
        ],
        selectedItemColor: Colors.deepPurple,
        unselectedItemColor: Colors.grey,
      ),
    );
  }

  Widget buildCadastro() {
    return SingleChildScrollView(
      padding: const EdgeInsets.all(20),
      child: Column(
        children: [
          const Text('Cadastrar Aluno', style: TextStyle(fontSize: 22, fontWeight: FontWeight.bold)),
          const SizedBox(height: 20),
          TextField(
            controller: nomeController,
            decoration: const InputDecoration(labelText: 'Nome do Aluno'),
          ),
          TextField(
            controller: frequenciaController,
            decoration: const InputDecoration(labelText: 'Frequência semanal (treinos)'),
            keyboardType: TextInputType.number,
          ),
          TextField(
            controller: atrasoController,
            decoration: const InputDecoration(labelText: 'Pagamentos em atraso'),
            keyboardType: TextInputType.number,
          ),
          TextField(
            controller: participacaoController,
            decoration: const InputDecoration(labelText: 'Participação (%) nas atividades'),
            keyboardType: TextInputType.number,
          ),
          const SizedBox(height: 20),
          ElevatedButton.icon(
            onPressed: adicionarAluno,
            icon: const Icon(Icons.save),
            label: const Text('Salvar Aluno'),
            style: ElevatedButton.styleFrom(
              backgroundColor: Colors.deepPurple,
              padding: const EdgeInsets.symmetric(horizontal: 20, vertical: 12),
            ),
          ),
        ],
      ),
    );
  }

  Widget buildLista() {
    if (alunos.isEmpty) {
      return const Center(child: Text('Nenhum aluno cadastrado.'));
    }

    final inverno = [6, 7, 8].contains(DateTime.now().month);

    return ListView.builder(
      padding: const EdgeInsets.all(12),
      itemCount: alunos.length + 1,
      itemBuilder: (context, index) {
        if (index == 0) {
          final total = alunos.length;
          final altoRisco = alunos.where((a) => a.riscoAlto).length;
          final baixoRisco = total - altoRisco;

          return Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Text('Total de Alunos: $total', style: const TextStyle(fontWeight: FontWeight.bold)),
              Text('Alunos com alto risco: $altoRisco', style: const TextStyle(color: Colors.red, fontWeight: FontWeight.bold)),
              Text('Alunos com baixo risco: $baixoRisco', style: const TextStyle(color: Colors.green, fontWeight: FontWeight.bold)),
              if (inverno) ...[
                const SizedBox(height: 10),
                const Text(
                  'ATENÇÃO: Estamos no INVERNO! A evasão costuma aumentar.',
                  style: TextStyle(color: Colors.deepPurple, fontWeight: FontWeight.bold),
                ),
              ],
              const Divider(thickness: 2),
            ],
          );
        }

        final aluno = alunos[index - 1];
        return Card(
          color: aluno.riscoAlto ? Colors.red.shade100 : Colors.green.shade100,
          child: ListTile(
            title: Text(aluno.nome, style: const TextStyle(fontWeight: FontWeight.bold)),
            subtitle: Text(
              'Frequência: ${aluno.frequenciaSemanal}x/sem  |  Atrasos: ${aluno.pagamentosAtrasados}  |  Participação: ${aluno.participacaoAtividades}%',
            ),
            trailing: Icon(
              aluno.riscoAlto ? Icons.warning : Icons.check_circle,
              color: aluno.riscoAlto ? Colors.red : Colors.green,
            ),
          ),
        );
      },
    );
  }
}
