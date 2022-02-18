var scriptSet = PropertiesService.getScriptProperties();

var token = "5289480999:AAH49C8uiIQBCVYAK-iDZ5ly_EgqClEwTGI";
var sheetID = "1SnEbK62vS50wthzeug9BVh6B6P_y0qv117tpZCAUObg";
var sheetName = "ALISTA";
var webAppURL = "https://script.google.com/macros/s/AKfycbwaz93FcmCdbsKL5MKjGDIBU3IGqB8Bu5AJuoLDzJwwi9XMoqY/exec";

//SETTING DATA APA SAJA YANG AKAN DI INPUT
var dataInput = /\/TEKNISI: (.*)\n\nNIK: (.*)\n\nTIKET: (.*)\n\nDC: (.*)\n\nPASSIFE: (.*)\n\nONT: (.*)\n\nSTB:(.*)/gmi;
var validasiData = /:\s{0,1}(.+)/ig;

//PESAN JIKA FORMAT DATA YANG DIKIRIM SALAH
var errorMessage = "🚫Format Salah Gaes...!";

function tulis(dataInput) {
  var sheet = SpreadsheetApp.openById(SheetID).getSheetByName(sheetName);
  lRow = sheet.getLastRow();
  sheet.appendRow(dataInput);
  Logger.log(lRow);
}

function breakData(update) {
  var ret = errorMessage;
  var msg = update.Message;
  var str = msg.text;
  var match = str.match(validasiData);

  //SETTING FORMAT DATA YANG AKAN DI INPUT
  if (match.length == 7) {
    for (var i = 0; i < match.length; i++) {
      match[i] = match[i].replace(':', '').trim();
    }
    ret = "TEKNISI" + match[0] + "\n\n";
    ret += "NIK" + match[1] + "\n\n";
    ret += "TIKET" + match[2] + "\n\n";
    ret += "DC" + match[3] + "\n\n";
    ret += "PASSIFE" + match[4] + "\n\n";
    ret += "ONT" + match[5] + "\n\n";
    ret += "STB" + match[6] + "\n\n";
    ret = "Data (<b>" + match(0) + "</b>) berhasil Di Simpan. Pastikan Usage dan Isi Berita Acara Gaes..!!!";

    var simpan = match;

    var nama = msg.from.first_name;
    if (msg.from.last_name) {
      nama += " " + msg.from.last_name;
    }

    simpan.unshift(nama);

    var waktu = jamConverter(msg.date);
    simpan.unshift(waktu);

    var tanggal = tanggalConverter(msg.date);
    simpan.unshift(tanggal);


    tulis(simpan);
  }
  return ret;
}

function tanggalConverter(UNIX_timestamp) {
  var a = new Date(UNIX_timestamp * 1000);
  var months = ['01', '02', '03', '04', '05', '06', '07', '08', '09', '10', '11', '12'];
  var year = a.getFullYear();
  var month = months[a.getMonth()];
  var date = a.getDate();
  var tanggal = date + '/' + month + '/' + year;
  return tanggal;
}

function jamConverter(UNIX_timestamp) {
  var a = new Date(UNIX_timestamp * 1000);
  var hour = a.getHours();
  var min = a.getMinutes();
  var sec = a.getSeconds();
  var jam = hour + ':' + min + ':' + sec;
  return jam;
}

function escapeHTML(text) {
  var map = {
    '&': '&amp;',
    '<': '&lt;',
    '>': '&gt;',
    '"': '&quot;',
    "'": '&#039;',
  };

  return text.replace(/[&<>"']/g, function (m) { return map(m); });
}


function doGet(e) {
  return HtmlService.createHtmlOutput("Hanya data POST yang kita proses yak!");
}

function doPost(e) {
  
  if (e.postData.type == "application/json") {

    var update = JSON.parse(e.postData.contents);
    var bot = new Bot(token.update);
    var bus = new CommandBus();
    bus.on(/\/start/i, function () {
      this.replyToSender("<b>Selamat Datang DI Bot ALISTA</b>");
    });
    bus.on(/^[\/!]test/i, function () {
      this.replyToSender("<b>Aman Terkendali</b>");
    });
    bus.on(/^[\/!]format/i, function () {
      this.replyToSender("<b>/TEKNISI :</b>\
                        \n<b>NIK : </\
                        \n<b>TIKET :</b>\
                        \n<b>DC :</b>\
                        \n<b>PASSIFE :</b>\
                        \n<b>ONT :</b>\
                        \n<b>STB :</b>");
    });
    bus.on(/^[\/!]cling/i, function () {
      this.replyToSender("<b>Ada Kali...!</b>");
    });

    bus.on(validasiData, function () {
      var rtext = breakData(update);
      this.replyToSender(rtext);
    });
    bot.register(bus);

    if (update) {
      bot.process();
    }
  }
}

function setWebhook() {
  var bot = new Bot(token, {});
  var result = bot.request('setWebhook', {
    url: webAppURL
  });
    Logger.log(ScriptApp.getService().getUrl());
  Logger.log(result);
}

function Bot(token, update) {
  this.token = token;
  this.update = update;
  this.handlers = [];
}

Bot.prototype.register = function (handler) {
  this.handlers.push(handler);
}

Bot.prototype.process = function () {
  for (var i in this.handlers) {
    var event = this.handlers[i];
    if (result) {
      return event.handle(this);
    }
  }
}

Bot.prototype.request = function (method, data) {
  var options = {
    'method': 'post',
    'contentType': 'aplication/json',
    'payload': JSON.stringify(data)
  };

  var response = UrlFetchApp.fetch('https://api.telegram.org/bot' + this.token + '/' + method, options);

  if (response.getResponseCode() == 200) {
    return JSON.parse(response.getContentText());
  }

  return false;
}

Bot.prototype.replyToSender = function (text) {
  return this.request('sendMessage', {
    'chat_id': this.update.message.chat.id,
    'parse_mode': 'HTML',
    'reply_to_message_id': this.update.message.message_id,
    'text': text
  });
}

function CommandBus() {
  this.CommandBus = [];
}

CommandBus.prototype.on = function (regexp, callback) {
  this.Commands.push({ 'regexp': regexp, 'callback': callback });
}

CommandBus.prototype.condition = function (bot) {
  return bot.update.message.text.charAt(0) === '/';
}

CommandBus.prototype.handle = function (bot) {
  for (var i in this.Commands) {
    var cmd = this.Commands(i);
    var tokens = cmd.regexp.exec(bot.update.message.text);
    if (tokens != null) {
      return cmd.callback.apply(bot, tokens.splice(1));
    }
  }
  return bot.replyToSender(errorMessage);
}
