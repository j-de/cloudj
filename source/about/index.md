---
title: about
date: 2019-06-01 13:39:07
---
You can contact me with this form but it's still in a beta mode. 


<script>
function submitToAPI(e) {
       e.preventDefault();
       var URL = "https://vgvgd2ah3g.execute-api.eu-west-3.amazonaws.com/production/contact";

            var Namere = /[A-Za-z]{1}[A-Za-z]/;
            if (!Namere.test($("#name-input").val())) {
                         alert ("Your name should be longer");
                return;
            }
            var mobilere = /[0-9]{10}/;
            if (!mobilere.test($("#phone-input").val())) {
                alert ("Please enter valid mobile number");
                return;
            }
            if ($("#email-input").val()=="") {
                alert ("Please enter your email");
                return;
            }

            var reeamil = /^([\w-\.]+@([\w-]+\.)+[\w-]{2,6})?$/;
            if (!reeamil.test($("#email-input").val())) {
                alert ("Please enter valid email address");
                return;
            }

       var name = $("#name-input").val();
       var phone = $("#phone-input").val();
       var email = $("#email-input").val();
       var desc = $("#description-input").val();
       var data = {
          name : name,
          phone : phone,
          email : email,
          desc : desc
        };

       $.ajax({
         type: "POST",
         url : "https://vgvgd2ah3g.execute-api.eu-west-3.amazonaws.com/production/contact",
         dataType: "json",
         crossDomain: "true",
         contentType: "application/json; charset=utf-8",
         data: JSON.stringify(data),

         
         success: function () {
           // clear form and show a success message
           alert("Your message has been sent! I'll contact you as soon as possible");
           document.getElementById("contact-form").reset();
       location.reload();
         },
         error: function () {
           // show an error message
           alert("There was a problem sending the message. Please try later or try to reach me through social media");
         }});
     }
</script>


<form id="contact-form" method="post">
      <h4>First Name:</h4>
      <input type="text" id="name-input" placeholder="Enter name here…" class="form-control" style="width:100%;" /><br/>
      <h4>Phone:</h4>
      <input type="phone"id="phone-input" placeholder="Enter phone number" class="form-control" style="width:100%;"/><br/>
      <h4>Email:</h4>
      <input type="email" id="email-input" placeholder="Enter email here…" class="form-control" style="width:100%;"/><br/>
      <h4>Your message:</h4>
      <textarea id="description-input" rows="10" placeholder="Enter your message…" class="form-control" style="width:100%;"></textarea><br/>
      <div class="g-recaptcha" data-sitekey="6LfFNqgUAAAAAHtz_FXiY2aeZe5u7-KVYlU3s-Wi"></div>
      <button type="button" onClick="submitToAPI(event)" class="btn btn-lg" style="margin-top:20px;">Submit</button>
</form>

