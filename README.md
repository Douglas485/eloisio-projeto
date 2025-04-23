const qrcode = require("qrcode-terminal");
const { Client } = require("whatsapp-web.js");

const client = new Client();
const delay = (ms) => new Promise((res) => setTimeout(res, ms));

const userState = {};

client.on("qr", (qr) => {
  qrcode.generate(qr, { small: true });
});

client.on("ready", () => {
  console.log("✅ Tudo certo! WhatsApp conectado.");
});

client.initialize();

client.on("message", async (msg) => {
  const chat = await msg.getChat();
  const userId = msg.from;

  if (msg.body.toLowerCase() === "menu" || msg.body.toLowerCase() === "voltar") {
    delete userState[userId];
    await client.sendMessage(
      userId,
      "🔄 Você voltou ao Menu Principal.\nComo posso ajudá-lo hoje?\n\n" +
        "1️⃣ Direito Trabalhista\n" +
        "2️⃣ Direito da Família\n" +
        "3️⃣ Consultar processo\n" +
        "4️⃣ Outros Assuntos"
    );
    return;
  }

  if (!userState[userId] && msg.body.match(/(menu|oi|olá|ola|bom dia|boa tarde|boa noite)/i)) {
    await chat.sendStateTyping();
    await delay(2000);

    const contact = await msg.getContact();
    const name = contact.pushname.split(" ")[0];

    userState[userId] = { step: "start" };

    await client.sendMessage(
      userId,
      `Olá, ${name}! O Dr. Eloisio Silva agradece seu contato. Como posso ajudá-lo hoje?\n\n` +
        "1️⃣ Direito Trabalhista\n" +
        "2️⃣ Direito da Família\n" +
        "3️⃣ Consultar processo\n" +
        "4️⃣ Outros Assuntos"
    );
    return;
  }

  if (!userState[userId]) return;

  const state = userState[userId];

  if (msg.body === "1" && state.step === "start") {
    state.step = "trabalhista_nome";
    await client.sendMessage(userId, "Qual é o seu nome por favor?");
    return;
  }

  if (msg.body === "2" && state.step === "start") {
    state.step = "familia_acao";
    await client.sendMessage(userId, "Sobre o que deseja falar?\n1️⃣ Divórcio\n2️⃣ Pensão\n3️⃣ Execução de Pensão\n4️⃣ Inventário");
    return;
  }

  if (msg.body === "3" && state.step === "start") {
    state.step = "processo_nome";
    await client.sendMessage(userId, "Informe o nome completo para iniciar a consulta.");
    return;
  }

  if (msg.body === "4" && state.step === "start") {
    state.step = "outros_assuntos";
    await client.sendMessage(userId, "Olá, descreva o assunto que deseja que em breve um de nossos assessores entrará em contato.");
    return;
  }

  // Fluxo Trabalhista
  if (state.step === "trabalhista_nome") {
    state.nome = msg.body;
    state.step = "trabalhista_trabalhando";
    await client.sendMessage(userId, "Atualmente está trabalhando?\n1️⃣ Sim\n2️⃣ Não");
    return;
  }

  
  if (state.step === "trabalhista_trabalhando") {
    state.trabalhando = msg.body === "1" ? "Sim" : "Não";
    state.step = state.trabalhando === "Sim" ? "trabalhista_empresa" : "trabalhista_data_admissao";
    await client.sendMessage(
      userId,
      state.trabalhando === "Sim"
        ? "Digite o nome da empresa onde trabalha:"
        : "Data de admissão/demissão?"
    );
    return;
  }

  if (state.step === "trabalhista_empresa") {
    state.empresa = msg.body;
    state.step = "trabalhista_data_admissao";
    await client.sendMessage(userId, "Data de admissão/demissão?");
    return;
  }

  if (state.step === "trabalhista_data_admissao") {
    state.data_admissao = msg.body;
    state.step = "trabalhista_funcao";
    await client.sendMessage(userId, "Qual era a sua função?");
    return;
  }

  if (state.step === "trabalhista_funcao") {
    state.funcao = msg.body;
    state.step = "trabalhista_salario";
    await client.sendMessage(userId, "Qual o seu salário?");
    return;
  }

  if (state.step === "trabalhista_salario") {
    state.salario = msg.body;
    state.step = "trabalhista_registro";
    await client.sendMessage(
      userId,
      "Trabalhava com registro em carteira?\nSe sim, informe o período com registro.\nSe não, informe o período sem registro."
    );
    return;
  }

  if (state.step === "trabalhista_registro") {
    state.registro = msg.body;
    state.step = "trabalhista_horario";
    await client.sendMessage(
      userId,
      "Relacione seu horário de trabalho e informe também os dias trabalhados."
    );
    return;
  }

  if (state.step === "trabalhista_horario") {
    state.horario = msg.body;
    state.step = "trabalhista_horas_extras";
    await client.sendMessage(
      userId,
      "Realiza horas extras? Você recebia horas extras ou era banco de horas?"
    );
    return;
  }

  if (state.step === "trabalhista_horas_extras") {
    state.horas_extras = msg.body;
    state.step = "trabalhista_adicional_noturno";
    await client.sendMessage(
      userId,
      "Recebia adicional noturno? Se não, deveria receber adicional noturno?"
    );
    return;
  }

  if (state.step === "trabalhista_adicional_noturno") {
    state.adicional_noturno = msg.body;
    state.step = "trabalhista_horario_almoco";
    await client.sendMessage(userId, "Fazia horário de almoço?");
    return;
  }

  if (state.step === "trabalhista_horario_almoco") {
    state.horario_almoco = msg.body;
    state.step = "trabalhista_atividade_periculosa";
    await client.sendMessage(
      userId,
      "Exercia atividade periculosa? Se sim, descreva sua atividade."
    );
    return;
  }

  if (state.step === "trabalhista_atividade_periculosa") {
    state.atividade_periculosa = msg.body;
    state.step = "trabalhista_vale_transporte";
    await client.sendMessage(userId, "Vale transporte: Recebia?");
    return;
  }

  if (state.step === "trabalhista_vale_transporte") {
    state.vale_transporte = msg.body;
    state.step = "trabalhista_vale_refeicao";
    await client.sendMessage(
      userId,
      "Vale refeição: Recebia? Levava marmita para o trabalho?"
    );
    return;
  }

  if (state.step === "trabalhista_vale_refeicao") {
    state.vale_refeicao = msg.body;
    state.step = "trabalhista_desconto_indevido";
    await client.sendMessage(userId, "Teve algum desconto indevido? Se sim, descreva:");
    return;
  }

  if (state.step === "trabalhista_desconto_indevido") {
    state.desconto_indevido = msg.body;
    state.step = "trabalhista_acidente_trabalho";
    await client.sendMessage(
      userId,
      "Teve alguma doença ou acidente de trabalho? Se sim, descreva:"
    );
    return;
  }

  if (state.step === "trabalhista_acidente_trabalho") {
    state.acidente_trabalho = msg.body;
    state.step = "trabalhista_direito";
    await client.sendMessage(
      userId,
      "Existe algum direito não mencionado nas perguntas acima? Se sim, descreva:"
    );
    return;
  }

  if (state.step === "trabalhista_direito") {
    state.direito = msg.body;
    state.step = "responder_enquete";
    await client.sendMessage(
    userId, "Obrigado por responder a enquete! Em breve entraremos em contato");
    delete userState[userId];
    return;

  }




 // Fluxo Família
if (state.step === "familia_acao") {
    const escolha = msg.body;
    switch (escolha) {
      case "1":
        state.acao = "divorcio";
        state.step = "divorcio_tipo";
        await client.sendMessage(userId, "O divórcio seria: Consensual (em comum acordo) ou Litigioso (sem acordo)?");
        break;
      case "2":
        state.acao = "pensao";
        state.step = "pensao_filhos";
        await client.sendMessage(userId, "A pensão é referente a quantos filhos?");
        break;
      case "3":
        state.acao = "execucao";
        state.step = "execucao_sentenca";
        await client.sendMessage(userId, "Existe uma sentença determinando o valor da pensão?");
        break;
      case "4":
        state.acao = "inventario";
        state.step = "inventario_acao";
        await client.sendMessage(userId, "A ação de inventário seria: Consensual ou Litigiosa?");
        break;
      case "5":
        state.acao = "consulta";
        state.step = "processo_nome";
        await client.sendMessage(userId, "Olá ! Nos informe seu nome completo por favor.");
        break;
      default:
        await client.sendMessage(userId, "Escolha uma opção válida:\n1️⃣ Divórcio\n2️⃣ Pensão\n3️⃣ Execução de Pensão\n4️⃣ Inventário\n5️⃣ Consultar Processo");
    }
    return;
  }
  
  // Fluxo Divórcio
  if (state.step === "divorcio_tipo") {
    state.divorcio_tipo = msg.body;
    state.step = "divorcio_filhos";
    await client.sendMessage(userId, "Tem filho menor de idade? Se sim, quantos?");
    return;
  }
  
  if (state.step === "divorcio_filhos") {
    state.divorcio_filhos = msg.body;
    state.step = "divorcio_consulta";
    await client.sendMessage(userId, "Tem bens a partilhar? Se sim, descreva os bens.");
    return;
  }

  if (state.step === "divorcio_consulta") {
    state.divorcio_consulta = msg.body;
    state.step = "divorcio_bens";
    await client.sendMessage(userId, "Deseja uma consulta com o Dr. Eloisio Silva?Se sim o valor da consulta é de $100 reais a ser pagos no ato da conferência. Com isso também enviaremos datas e horários disponíveis.).");
    return;
  }
  
  if (state.step === "divorcio_bens") {
    state.divorcio_bens = msg.body;
    await client.sendMessage(userId, "Obrigado por responder a enquete! Em breve entraremos em contato.");
    delete userState[userId];
    return;
  }
  
  // Fluxo Pensão
  if (state.step === "pensao_filhos") {
    state.pensao_filhos = msg.body;
    state.step = "pensao_autonomo";
    await client.sendMessage(userId, "O devedor possui emprego com carteira assinada ou é autonomo? E se possível forneça os dados da empresa.");
    return;
  }
  
  
  if (state.step === "pensao_autonomo") {
    state.pensao_autonomo = msg.body;
    state.step = "pensao_consulta";
    await client.sendMessage(userId, "Deseja entrar com uma ação para receber a pensão? Caso deseje, seguem formas de pagamento: à vista ou parcelas mínimas de 500 reais, seja no cartão de crédito ou boleto.");
    return;
  }
  
  if (state.step === "pensao_consulta") {
    state.pensao_consulta = msg.body.toLowerCase();
    
    if (state.pensao_consulta === "sim") {
      state.step = "pensao_agendamento";
      await client.sendMessage(userId, "Gostaria de realizar uma vídeo conferência ou chamada de voz para sanar possíveis dúvidas? Em breve entraremos em contato disponibilizando datas e horários para realizarmos uma vídeo conferência.");
      return;
    } else {
      state.step = "pensao_acao";
      await client.sendMessage(userId, "Gostaria de realizar uma vídeo conferência ou chamada de voz para sanar possíveis dúvidas? Em breve entraremos em contato disponibilizando datas e horários para realizarmos uma vídeo conferência");
      return;
    }
  }
  
  if (state.step === "pensao_acao") {
    state.pensao_acao = msg.body;
    await client.sendMessage(userId, "Obrigado por responder a enquete! Em breve um de nossos assessores entrará em contato. O Dr. Eloisio Silva agradece o seu contato.");
    delete userState[userId];
    return;
  }
  
  
  // Fluxo Execução de Pensão
  if (state.step === "execucao_sentenca") {
    state.execucao_sentenca = msg.body;
    state.step = "execucao_parcelas";
    await client.sendMessage(userId, "Há quanto tempo o devedor está inadimplente com o pagamento da pensão?");
    return;
}

if (state.step === "execucao_parcelas") {
    state.execucao_parcelas = msg.body;
    state.step = "execucao_inadimplente";
    await client.sendMessage(userId, "O valor desta ação custa $5.000 reais à vista ou com parcelas mínimas de $500 reais via boleto ou cartão de crédito. Deseja entrar com uma ação?");
    return;
}

if (state.step === "execucao_inadimplente") {
    state.execucao_inadimplente = msg.body;
    
    if (msg.body.toLowerCase() === "sim") {
        state.step = "execucao_video_chamada";
        await client.sendMessage(userId, "Gostaria de realizar uma vídeo conferência ou chamada de voz para sanar possíveis dúvidas? Em breve entraremos em contato disponibilizando datas e horários para realizarmos uma vídeo conferência.");
    } else {
        state.step = "execucao_encerramento";
        await client.sendMessage(userId, "Deseja uma consulta com o Dr. Eloisio Silva?Se sim o valor da consulta é de $100 reais a ser pagos no ato da conferência. Com isso também enviaremos datas e horários disponíveis..");
    }
    return;
}

if (state.step === "execucao_video_chamada" || state.step === "execucao_encerramento") {
    state.execucao_contrato = msg.body;
    await client.sendMessage(
        userId,
        "Obrigado por responder à enquete! Em breve um de nossos assessores entrará em contato. O Dr. Eloisio Silva agradece o seu contato."
    );
delete state.step;
}

  
  
  // Fluxo Inventário
  if (state.step === "inventario_acao") {
    state.inventario_acao = msg.body;
    state.step = "inventario_herdeiro";
    await client.sendMessage(userId, "Existe herdeiro menor de idade? Se sim quantos seriam");
    return;
  }
  
  if (state.step === "inventario_herdeiro") {
    state.inventario_herdeiro = msg.body;
    state.step = "inventario_num_herdeiros";
    await client.sendMessage(userId, "A ação envolve quantos herdeiros? E quais seriam os bens a inventariar?");
    return;
  }
  
  if (state.step === "inventario_num_herdeiros") {
    state.inventario_num_herdeiros = msg.body;
    state.step = "inventario_agendar";
    await client.sendMessage(userId, "Gostaria de agendar uma vídeo conferência com o Dr. Eloisio Silva?)");
    return;
  }
  
  if (state.step === "inventario_agendar") {
    state.inventario_agendar = msg.body;
    await client.sendMessage(userId, "Em breve um de nossos acessores entrará em contato. O Dr. Eloisio Silva agradece o seu contato.");
    delete userState[userId];
    return;
  }
  
  // Fluxo Consulta de Processo
  if (state.step === "processo_nome") {
    state.processo_nome = msg.body;
    state.step = "processo_consulta";
    await client.sendMessage(userId, "Por favor, forneça também o seu CPF e se pssível o numero do processo para realizarmos a consulta.");
    return;
  }
  
  if (state.step === "processo_consulta") {
    state.processo_consulta = msg.body;
    await client.sendMessage(userId, "Obrigado! Em breve, entraremos em contato para dar continuidade ao seu processo.");
    delete userState[userId];
    return;
  };
})
  ;
