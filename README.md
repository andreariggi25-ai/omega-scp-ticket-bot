
require('dotenv').config();
const {
  Client,
  GatewayIntentBits,
  Events,
  ChannelType,
  PermissionFlagsBits,
  ActionRowBuilder,
  ButtonBuilder,
  ButtonStyle,
  EmbedBuilder,
  ModalBuilder,
  TextInputBuilder,
  TextInputStyle,
} = require('discord.js');

const client = new Client({
  intents: [
    GatewayIntentBits.Guilds,
    GatewayIntentBits.GuildMembers,
  ],
});

const TOKEN = process.env.DISCORD_TOKEN;

// CATEGORIE
const CATEGORY_IDS = {
  ticketDiscord: '1544634601019023410',
  ticketSupportoMuted: '1544634734167195649',
  ticketPartnership: '1544634806925529088',
  ticketCandidature: '1544634866560139304',
  ticketOwner: '1544634908113244220',
};

client.once(Events.ClientReady, async (c) => {
  await c.application.commands.set([
    {
      name: 'ticketpanel',
      description: 'Invia il pannello dei ticket OMEGA-SCP',
    },
  ]);

  console.log(`Bot online come ${c.user.tag}`);
});


// ⭐ COMANDO /ticketpanel — 5 MESSAGGI SEPARATI
client.on(Events.InteractionCreate, async (interaction) => {
  try {
    if (interaction.isChatInputCommand() && interaction.commandName === 'ticketpanel') {

      await interaction.reply({ content: "📨 **Pannello Ticket OMEGA-SCP**", ephemeral: false });

      // 1 — Assistenza Tecnica (BLU)
      const embed1 = new EmbedBuilder()
        .setTitle('🛠️ Assistenza Tecnica')
        .setDescription(
          'Hai riscontrato un problema tecnico, un bug o un malfunzionamento all’interno del server OMEGA‑SCP?\n' +
          'Il nostro team è pronto ad assisterti con diagnosi, verifiche e soluzioni rapide.\n' +
          'Apri un ticket e descrivi nel dettaglio ciò che è accaduto: più informazioni fornisci, più velocemente potremo aiutarti.'
        )
        .setColor(0x5865f2); // BLU

      const row1 = new ActionRowBuilder().addComponents(
        new ButtonBuilder()
          .setCustomId('ticket:discord')
          .setLabel('Apri')
          .setStyle(ButtonStyle.Primary)
      );

      await interaction.channel.send({ embeds: [embed1], components: [row1] });

      // 2 — Supporto Muted (ROSSO)
      const embed2 = new EmbedBuilder()
        .setTitle('🔇 Supporto Muted')
        .setDescription(
          'Hai ricevuto un mute e ritieni che la sanzione sia stata applicata in modo errato, oppure desideri chiarimenti sulle motivazioni?\n' +
          'Il nostro staff analizzerà la situazione, controllerà le evidenze e valuterà il tuo caso con attenzione.\n' +
          'Apri un ticket e fornisci una spiegazione completa dell’accaduto.'
        )
        .setColor(0xff0000); // ROSSO

      const row2 = new ActionRowBuilder().addComponents(
        new ButtonBuilder()
          .setCustomId('ticket:muted')
          .setLabel('Apri')
          .setStyle(ButtonStyle.Primary)
      );

      await interaction.channel.send({ embeds: [embed2], components: [row2] });

      // 3 — Partnership (GIALLO)
      const embed3 = new EmbedBuilder()
        .setTitle('🤝 Partnership')
        .setDescription(
          'Sei un content creator, gestisci un progetto o rappresenti una community e desideri collaborare con OMEGA‑SCP?\n' +
          'Siamo sempre aperti a valutare proposte di partnership, collaborazioni, eventi o iniziative congiunte.\n' +
          'Apri un ticket e presentaci la tua idea in modo dettagliato: il nostro team la esaminerà con professionalità.'
        )
        .setColor(0xffd700); // GIALLO

      const row3 = new ActionRowBuilder().addComponents(
        new ButtonBuilder()
          .setCustomId('ticket:partnership')
          .setLabel('Apri')
          .setStyle(ButtonStyle.Primary)
      );

      await interaction.channel.send({ embeds: [embed3], components: [row3] });

      // 4 — Candidature Staff (BIANCO)
      const embed4 = new EmbedBuilder()
        .setTitle('📝 Candidature Staff')
        .setDescription(
          'Hai superato la fase scritta del bando e sei pronto per sostenere l’orale?\n' +
          'Apri un ticket per fissare la data del colloquio e completare il processo di selezione.\n' +
          'Assicurati di includere tutte le informazioni richieste per velocizzare la procedura.'
        )
        .setColor(0xffffff); // BIANCO

      const row4 = new ActionRowBuilder().addComponents(
        new ButtonBuilder()
          .setCustomId('ticket:candidature')
          .setLabel('Apri')
          .setStyle(ButtonStyle.Primary)
      );

      await interaction.channel.send({ embeds: [embed4], components: [row4] });

      // 5 — Owner (ROSSO)
      const embed5 = new EmbedBuilder()
        .setTitle('👑 Owner – Richieste Riservate')
        .setDescription(
          'Questa sezione è dedicata esclusivamente alle comunicazioni dirette con il Founder / Owner di OMEGA‑SCP.\n' +
          'Utilizzala solo per questioni di massima priorità, richieste amministrative o comunicazioni riservate.\n' +
          'Apri un ticket solo se strettamente necessario.'
        )
        .setColor(0xff0000); // ROSSO

      const row5 = new ActionRowBuilder().addComponents(
        new ButtonBuilder()
          .setCustomId('ticket:owner')
          .setLabel('Apri')
          .setStyle(ButtonStyle.Primary)
      );

      await interaction.channel.send({ embeds: [embed5], components: [row5] });

      return;
    }

    // ⭐ PULSANTI
    if (interaction.isButton()) {
      const id = interaction.customId;

      if (id.startsWith('ticket:')) {
        if (
          id === 'ticket:discord' ||
          id === 'ticket:muted' ||
          id === 'ticket:partnership' ||
          id === 'ticket:candidature' ||
          id === 'ticket:owner'
        ) {
          return handleOpenTicket(interaction, id);
        }

        if (id === 'ticket:claim') return handleClaim(interaction);
        if (id === 'ticket:close') return handleClose(interaction);
        if (id === 'ticket:add') return showAddUserModal(interaction);
        if (id === 'ticket:remove') return showRemoveUserModal(interaction);
        if (id === 'ticket:rename') return showRenameModal(interaction);
      }
    }

    // ⭐ MODAL
    if (interaction.isModalSubmit()) {
      const id = interaction.customId;

      if (id === 'ticket:addmodal') return handleAddUser(interaction);
      if (id === 'ticket:removemodal') return handleRemoveUser(interaction);
      if (id === 'ticket:renamemodal') return handleRename(interaction);
    }

  } catch (err) {
    console.error(err);
  }
});


// ⭐ APERTURA TICKET
async function handleOpenTicket(interaction, buttonId) {
  const user = interaction.user;
  const guild = interaction.guild;

  const existing = guild.channels.cache.find(
    (ch) =>
      ch.type === ChannelType.GuildText &&
      ch.topic === `ticket:${user.id}:${buttonId}`
  );

  if (existing) {
    return interaction.reply({
      content: `Hai già un ticket aperto: ${existing}`,
      ephemeral: true,
    });
  }

  let parentId = null;
  let label = '';

  switch (buttonId) {
    case 'ticket:discord':
      parentId = CATEGORY_IDS.ticketDiscord;
      label = '🛠️ Assistenza Tecnica';
      break;

    case 'ticket:muted':
      parentId = CATEGORYORY_IDS.ticketSupportoMuted;
      label = '🔇 Supporto Muted';
      break;

    case 'ticket:partnership':
      parentId = CATEGORY_IDS.ticketPartnership;
      label = '🤝 Partnership';
      break;

    case 'ticket:candidature':
      parentId = CATEGORY_IDS.ticketCandidature;
      label = '📝 Candidature Staff';
      break;

    case 'ticket:owner':
      parentId = CATEGORY_IDS.ticketOwner;
      label = '👑 Owner';
      break;
  }

  const channel = await guild.channels.create({
    name: `ticket-${user.username}`.toLowerCase(),
    type: ChannelType.GuildText,
    parent: parentId,
    topic: `ticket:${user.id}:${buttonId}`,
    permissionOverwrites: [
      {
        id: guild.roles.everyone.id,
        deny: [PermissionFlagsBits.ViewChannel],
      },
      {
        id: user.id,
        allow: [
          PermissionFlagsBits.ViewChannel,
          PermissionFlagsBits.SendMessages,
          PermissionFlagsBits.ReadMessageHistory,
        ],
      },
      {
        id: client.user.id,
        allow: [
          PermissionFlagsBits.ViewChannel,
          PermissionFlagsBits.SendMessages,
          PermissionFlagsBits.ReadMessageHistory,
          PermissionFlagsBits.ManageChannels,
        ],
      },
    ],
  });

  const embed = new EmbedBuilder()
    .setTitle('📝 Nuovo Ticket Aperto')
    .setDescription(`Utente: ${user}\nCategoria: ${label}\n\nScrivi qui sotto i dettagli della tua richiesta.`)
    .setColor(0x5865f2);

  await channel.send({
    content: `<@${user.id}>`,
    embeds: [embed],
    components: buildTicketManageComponents(),
  });

  await interaction.reply({
    content: `Ticket creato: ${channel}`,
    ephemeral: true,
  });
}


// ⭐ PULSANTI DI GESTIONE
function buildTicketManageComponents() {
  return [
    new ActionRowBuilder().addComponents(
      new ButtonBuilder().setCustomId('ticket:claim').setLabel('Claim').setStyle(ButtonStyle.Success),
      new ButtonBuilder().setCustomId('ticket:close').setLabel('Chiudi').setStyle(ButtonStyle.Danger),
      new ButtonBuilder().setCustomId('ticket:add').setLabel('Aggiungi').setStyle(ButtonStyle.Secondary),
      new ButtonBuilder().setCustomId('ticket:remove').setLabel('Rimuovi').setStyle(ButtonStyle.Secondary),
      new ButtonBuilder().setCustomId('ticket:rename').setLabel('Rinomina').setStyle(ButtonStyle.Primary),
    ),
  ];
}


// ⭐ CLAIM
async function handleClaim(interaction) {
  await interaction.channel.send(`✅ Ticket claimato da ${interaction.user}`);
  await interaction.reply({ content: 'Hai claimato il ticket.', ephemeral: true });
}


// ⭐ CHIUDI
async function handleClose(interaction) {
  await interaction.reply({ content: 'Il ticket verrà chiuso...', ephemeral: true });
  setTimeout(() => interaction.channel.delete().catch(() => {}), 2000);
}


// ⭐ MODAL AGGIUNGI
async function showAddUserModal(interaction) {
  const modal = new ModalBuilder()
    .setCustomId('ticket:addmodal')
    .setTitle('Aggiungi utente');

  const input = new TextInputBuilder()
    .setCustomId('user_input')
    .setLabel('ID o menzione')
    .setStyle(TextInputStyle.Short)
    .setRequired(true);

  modal.addComponents(new ActionRowBuilder().addComponents(input));
  await interaction.showModal(modal);
}


// ⭐ MODAL RIMUOVI
async function showRemoveUserModal(interaction) {
  const modal = new ModalBuilder()
    .setCustomId('ticket:removemodal')
    .setTitle('Rimuovi utente');

  const input = new TextInputBuilder()
    .setCustomId('user_input')
    .setLabel('ID o menzione')
    .setStyle(TextInputStyle.Short)
    .setRequired(true);

  modal.addComponents(new ActionRowBuilder().addComponents(input));
  await interaction.showModal(modal);
}


// ⭐ MODAL RINOMINA
async function showRenameModal(interaction) {
  const modal = new ModalBuilder()
    .setCustomId('ticket:renamemodal')
    .setTitle('Rinomina ticket');

  const input = new TextInputBuilder()
    .setCustomId('name_input')
    .setLabel('Nuovo nome')
    .setStyle(TextInputStyle.Short)
    .setRequired(true);

  modal.addComponents(new ActionRowBuilder().addComponents(input));
  await interaction.showModal(modal);
}


// ⭐ AGGIUNGI UTENTE
async function handleAddUser(interaction) {
  const value = interaction.fields.getTextInputValue('user_input');
  const userId = extractUserId(value);

  if (!userId) {
    return interaction.reply({ content: 'Utente non valido.', ephemeral: true });
  }

  const member = await interaction.guild.members.fetch(userId).catch(() => null);
  if (!member) {
    return interaction.reply({ content: 'Utente non trovato.', ephemeral: true });
  }

  await interaction.channel.permissionOverwrites.edit(member.id, {
    ViewChannel: true,
    SendMessages: true,
    ReadMessageHistory: true,
  });

  await interaction.reply({
    content: `Aggiunto ${member} al ticket.`,
    ephemeral: true,
  });
}


// ⭐ RIMUOVI UTENTE
async function handleRemoveUser(interaction) {
  const value = interaction.fields.getTextInputValue('user_input');
  const userId = extractUserId(value);

  if (!userId) {
    return interaction.reply({ content: 'Utente non valido.', ephemeral: true });
  }

  const member = await interaction.guild.members.fetch(userId).catch(() => null);
  if (!member) {
    return interaction.reply({ content: 'Utente non trovato.', ephemeral: true });
  }

  await interaction.channel.permissionOverwrites.edit(member.id, {
    ViewChannel: false,
    SendMessages: false,
    ReadMessageHistory: false,
  });

  await interaction.reply({
    content: `Rimosso ${member} dal ticket.`,
    ephemeral: true,
  });
}


// ⭐ RINOMINA
async function handleRename(interaction) {
  const newName = interaction.fields.getTextInputValue('name_input');
  await interaction.channel.setName(newName);
  await interaction.reply({
    content: `Canale rinominato in **${newName}**.`,
    ephemeral: true,
  });
}


// ⭐ ESTRAI ID
function extractUserId(text) {
  const mention = text.match(/^<@!?(\d+)>$/);
  if (mention) return mention[1];

  const id = text.match(/^(\d{10,})$/);
  if (id) return id[1];

  return null;
}

client.login(TOKEN);
