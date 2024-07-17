.hidden {
  display: none;
}

#customQueryForm {
  padding: 20px;
  background-color: #f9f9f9;
  border: 1px solid #ccc;
  border-radius: 10px;
  margin: 20px 0;
}

#customQueryForm label {
  display: block;
  margin-bottom: 10px;
}

#customQueryForm button {
  margin-top: 20px;
  padding: 10px 20px;
  background-color: #007BFF;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

button.btn {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100%;
}

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Chatbot Widget</title>
    <!-- Google Fonts -->
    <link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet">
    <link href="https://fonts.googleapis.com/css2?family=Open+Sans&family=Raleway:500&family=Roboto:wght@300&family=Lato&display=swap" rel="stylesheet">
    <!-- Font Awesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/4.7.0/css/font-awesome.min.css" integrity="sha256-eZrrJcwDc/3uDhsdt61sL2oOBY362qM3lon1gyExkL0=" crossorigin="anonymous">
    <!-- Materialize CSS -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/materialize/1.0.0/css/materialize.min.css">
    <!-- Main CSS -->
    <link rel="stylesheet" href="static/css/style.css">
</head>
<body>
    <div class="container">
        <!-- Modal for rendering charts -->
        <div id="modal1" class="modal">
            <canvas id="modal-chart"></canvas>
        </div>
        <!-- Chatbot widget -->
        <div class="widget">
            <div class="chat_header">
                <span class="chat_header_title">Sara</span>
                <span class="dropdown-trigger" href="#" data-target="dropdown1">
                    <i class="material-icons">more_vert</i>
                </span>
                <!-- Dropdown menu -->
                <ul id="dropdown1" class="dropdown-content">
                    <li><a href="#" id="clear">Clear</a></li>
                    <li><a href="#" id="restart">Restart</a></li>
                    <li><a href="#" id="close">Close</a></li>
                </ul>
            </div>
            <div class="chats" id="chats">
                <div class="clearfix"></div>
            </div>
            <div class="keypad">
                <textarea id="userInput" placeholder="Type a message..." class="usrInput"></textarea>
                <div id="sendButton" role="button" aria-label="Send message">
                    <i class="fa fa-paper-plane" aria-hidden="true"></i>
                </div>
                <div id="customQueryButton" role="button" aria-label="Custom Query">
                    <i class="fa fa-search" aria-hidden="true"></i>
                </div>
            </div>
        </div>
        <!-- Custom Query Form -->
        <div id="customQueryForm" class="hidden">
            <form id="attributeForm">
                <p>
                    <label>
                        <input type="checkbox" name="attribute_1" />
                        <span>Attribute 1</span>
                    </label>
                </p>
                <p>
                    <label>
                        <input type="checkbox" name="attribute_2" />
                        <span>Attribute 2</span>
                    </label>
                </p>
                <!-- Add more attributes as needed -->
                <button type="button" onclick="submitForm()" class="btn">Submit</button>
            </form>
        </div>
        <!-- Bot profile -->
        <div class="profile_div" id="profile_div">
            <img class="imgProfile" src="static/img/botAvatar.png" alt="Bot Avatar">
        </div>
    </div>
    <!-- Scripts -->
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/materialize/1.0.0/js/materialize.min.js"></script>
    <script src="static/js/script.js"></script>
    <script src="static/js/lib/chart.min.js"></script>
    <script src="static/js/lib/showdown.min.js"></script>
    <script>
        document.addEventListener('DOMContentLoaded', function() {
            M.AutoInit();
        });
    </script>
</body>
</html>

document.addEventListener("DOMContentLoaded", function() {
 
  const elemsTap = document.querySelector(".tap-target");
  if (elemsTap) {
    const instancesTap = M.TapTarget.init(elemsTap, {});
    instancesTap.open();
    setTimeout(function() {
      instancesTap.close();
    }, 4000);
  }
  
  $("div").removeClass("tap-target-origin");
  $(".dropdown-trigger").dropdown();
  $(".modal").modal();
});

function include(file) {
  const script = document.createElement('script');
  script.src = file;
  script.type = 'text/javascript';
  script.defer = true;
  document.getElementsByTagName('head').item(0).appendChild(script);
}

include('./static/js/components/index.js');

window.addEventListener('load', function() {

  $(document).ready(function() {
    $("div").removeClass("tap-target-origin");
    $(".dropdown-trigger").dropdown();
    $(".modal").modal();
  });

  $("#profile_div").click(function() {
    $(".profile_div").toggle();
    $(".widget").toggle();
  });

  $("#clear").click(function() {
    $(".chats").fadeOut("normal", function() {
      $(".chats").html("");
      $(".chats").fadeIn();
    });
  });

  $("#close").click(function() {
    $(".profile_div").toggle();
    $(".widget").toggle();
    scrollToBottomOfResults();
  });
  $('#customQueryButton').click(function() {
    $('#customQueryForm').toggleClass('hidden');
});

// Submit custom query form
async function submitForm() {
    const form = document.getElementById('attributeForm');
    const formData = new FormData(form);
    const attributes = {};

    formData.forEach((value, key) => {
        attributes[key] = value === 'on';
    });

    const response = await fetch('http://localhost:5005/webhooks/rest/webhook', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
            sender: 'user',
            message: JSON.stringify(attributes)
        }),
    });

    const data = await response.json();
    console.log(data);
    setBotResponse(data);

    // Hide the form after submission
    $('#customQueryForm').addClass('hidden');
}

// Function to send user message to Rasa server
function sendCustomQuery(query) {
    $.ajax({
        url: 'http://localhost:5005/webhooks/rest/webhook',
        type: 'POST',
        contentType: 'application/json',
        data: JSON.stringify({ message: query, sender: sender_id }),
        success: function(botResponse, status) {
            console.log('Response from Rasa:', botResponse, 'Status:', status);
            setBotResponse(botResponse);
        },
        error: function(xhr, textStatus) {
            console.log('Error from bot end:', textStatus);
            setBotResponse([]);
        }
    });
}
});

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Chatbot Widget</title>
    <!-- Google Fonts -->
    <link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet">
    <link href="https://fonts.googleapis.com/css2?family=Open+Sans&family=Raleway:500&family=Roboto:wght@300&family=Lato&display=swap" rel="stylesheet">
    <!-- Font Awesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/4.7.0/css/font-awesome.min.css" integrity="sha256-eZrrJcwDc/3uDhsdt61sL2oOBY362qM3lon1gyExkL0=" crossorigin="anonymous">
    <!-- Materialize CSS -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/materialize/1.0.0/css/materialize.min.css">
    <!-- Main CSS -->
    <link rel="stylesheet" href="static/css/style.css">
</head>
<body>
    <div class="container">
        <!-- Modal for rendering charts -->
        <div id="modal1" class="modal">
            <canvas id="modal-chart"></canvas>
        </div>
        <!-- Chatbot widget -->
        <div class="widget">
            <div class="chat_header">
                <span class="chat_header_title">Sara</span>
                <span class="dropdown-trigger" href="#" data-target="dropdown1">
                    <i class="material-icons">more_vert</i>
                </span>
                <!-- Dropdown menu -->
                <ul id="dropdown1" class="dropdown-content">
                    <li><a href="#" id="clear">Clear</a></li>
                    <li><a href="#" id="restart">Restart</a></li>
                    <li><a href="#" id="close">Close</a></li>
                </ul>
            </div>
            <div class="chats" id="chats">
                <div class="clearfix"></div>
            </div>
            <div class="keypad">
                <textarea id="userInput" placeholder="Type a message..." class="usrInput"></textarea>
                <div id="sendButton" role="button" aria-label="Send message">
                    <i class="fa fa-paper-plane" aria-hidden="true"></i>
                </div>
                <div id="customQueryButton" role="button" aria-label="Custom Query">
                    <i class="fa fa-search" aria-hidden="true"></i>
                </div>
            </div>
        </div>
        <!-- Custom Query Form -->
        <div id="customQueryForm" class="hidden">
            <form id="attributeForm">
                <p>
                    <label>
                        <input type="checkbox" name="attribute_1" />
                        <span>Attribute 1</span>
                    </label>
                </p>
                <p>
                    <label>
                        <input type="checkbox" name="attribute_2" />
                        <span>Attribute 2</span>
                    </label>
                </p>
                <!-- Add more attributes as needed -->
                <button type="button" onclick="submitForm()" class="btn">Submit</button>
            </form>
        </div>
        <!-- Bot profile -->
        <div class="profile_div" id="profile_div">
            <img class="imgProfile" src="static/img/botAvatar.png" alt="Bot Avatar">
        </div>
    </div>
    <!-- Scripts -->
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/materialize/1.0.0/js/materialize.min.js"></script>
    <script src="static/js/script.js"></script>
    <script src="static/js/lib/chart.min.js"></script>
    <script src="static/js/lib/showdown.min.js"></script>
    <script>
        document.addEventListener('DOMContentLoaded', function() {
            M.AutoInit();
        });
    </script>
</body>
</html>



