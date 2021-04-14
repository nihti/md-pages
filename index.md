# Tervetuloa

Palvelinten hallinnan kurssimateriaalit: <https://terokarvinen.com/2021/configuration-management-systems-palvelinten-hallinta-ict4tn022-spring-2021/>

## Ensimmäinen tehtävä

Ensimmäinen kurssitehtävä oli luoda tämä sivusto ja opiskella Linuxin kansiorakenne ja tavallisimpia komentoja. Lisäksi tuli asentaa master-minion -arkkitehtuuri. Seurataan ohjeita: <http://terokarvinen.com/2018/salt-quickstart-salt-stack-master-and-slave-on-ubuntu-linux/>

### Masterin asennus
    sudo apt-get update
    sudo apt-get -y install salt-master
    hostname -I 
    
Hostname kertoo että Masterin IP on 10.0.2.15

### Minionin asennus
    sudo apt-get update
    sudo apt-get install salt-minion
    
Koska en käyttänyt -y flagia, joudun hyväksymään "Do you want to continue?" - y
    
    sudoedit /etc/salt/minion

Lisätään rivit missä kerrotaan Minionille Masterin tiedot:
    
    master: 10.0.2.15
    id: nihti-VirtualBox

Käynnistetään Minion uudestaan että se ottaa muutokset käyttöön:

    sudo systemctl restart salt-minion.service

Hyväksytään Masterin avain:

    sudo salt-key -A

![kuva](https://user-images.githubusercontent.com/22195470/114585605-7d82a300-9c8c-11eb-860b-56eabd6db6a5.png)

Kokeillaan Minionin komentamista:

    sudo salt '*' cmd.run 'whoami'

![kuva](https://user-images.githubusercontent.com/22195470/114585855-be7ab780-9c8c-11eb-8cab-4a8e1f068b33.png)

Kokeillaan muita komentoja ohjeissa, cmd.run 'hostname -I' ja grains.items komennot toimivat, mutta 

    sudo salt '*' pkg.install httpie
    
kaataa virtuaalikoneen. Virtuaalikone vaikutti jo aiemmin kaatuvan muutaman kerran, joten komennolla ei ole välttämättä yhteyttä tähän. Kokeillaan uudestaan ja saadaan tulos:

![kuva](https://user-images.githubusercontent.com/22195470/114587541-7197e080-9c8e-11eb-9013-26b26a01b1c5.png)

Varmistetaan että salt toimii. Kokeillaan jo toiminutta komentoa.

    sudo salt '*' cmd.run 'hostname -I' 
    
![kuva](https://user-images.githubusercontent.com/22195470/114588283-2d591000-9c8f-11eb-97f8-ba8daa98b58b.png)

Siirrytään seuraaviin vaiheisiin.

### Ohjeita Minioneille Masterilta

Seurataan toista ohjetta: <http://terokarvinen.com/2018/salt-states-i-want-my-computers-like-this/>
Luodaan kansio Masterin ohjeita varten

    sudo mkdir -p /srv/salt/

Luodaan tiedosto hello.sls ja sinne rivit

    /tmp/helloworld.txt
      file.managed:
        - source: salt://helloworld.txt

Polku salt://helloworld.txt tarkoittaa Masterin /srv/salt kansiota. 
Lisätään tiedosto helloworld.txt

    sudoedit /srv/salt/helloworld.txt
    
Ja sinne rivit

    Hello World!

Yritetään ajaa tiedosto hello.sls:

    sudo salt '*' state.apply hello
    
![kuva](https://user-images.githubusercontent.com/22195470/114594451-ef131f00-9c95-11eb-9db2-b7d0bc9c8812.png)

Herjaa että rivillä 2 ongelma. Huomataan että rivillä 1 unohtunut kaksoispiste rivin lopusta. Korjataan kaksoispiste ja ajetaan komento uudestaan:

![kuva](https://user-images.githubusercontent.com/22195470/114594628-21248100-9c96-11eb-9703-f5b8de48889f.png)

Toimii. Kokeillaan seuraavaksi lisätä automaattisesti puolentunnin välein ajettava top.sls:

    sudoedit /srv/salt/top.sls
    
Lisätään rivit:
    
    base:
      '*'
        - hello

Kokeillaan ajaa tiedosto välittömästi:

    sudo salt '*' state.highstate

![kuva](https://user-images.githubusercontent.com/22195470/114594983-8aa48f80-9c96-11eb-93c8-6e9a2040b04b.png)

Jälleen oli kaksoispiste unohtunut tiedostosta vaikkei se tällä kertaa sitä harjannutkaan, nyt toiselta riviltä. Lisätään kaksoispiste ja kokeillaan uudestaan:

![kuva](https://user-images.githubusercontent.com/22195470/114596723-a01ab900-9c98-11eb-8650-564493c71f29.png)

Lopetetaan tehtävä.


## Toinen tehtävä

Tutustutaan configuration management systemiin ja package-file-service patterniin. Asennetaan ohjelmisto, korvataan konfiguraatiotiedosto ja käynnistetään daemon uudestaan uudella konfiguraatiolla. Varsinaisia ohjeita ssh:n asentamiseen kurssimateriaaleista ei löydy eli ilmeisesti kyseessä on itsestäänselvyys. Etsitään kuitenkin varmuudeksi joku lähde ssh-serverin asentamiseksi: <https://www.cyberciti.biz/faq/ubuntu-linux-install-openssh-server/>

Aloitetaan komennoilla:
        
    sudo apt-get update
    sudo apt-get install openssh-server

VirtualBoxin virtuaalikoneet ovat vaikuttaneet epävakailta ja jälleen uuden ohjelmiston asentaminen vaikuttaa kaataneen koneen. Kone ei ollut kaatunut mutta asentaminen vei aikaa ja kirjasi minut ulos Ubuntusta. 

Jatketaan:

    sudo systemctl enable ssh
    sudo systemctl start ssh
    
Luodaan SSH state:

    openssh-server:
      pkg.installed
    /etc/ssh/sshd_config:
      file.managed:
        - source: salt://sshd_config
      sshd:
        service.running:
          - watch:
            - file: /etc/ssh/sshd_config


## Apachen asennus tunnilla

sudo apt-get -y install apache2
sudo a2enmod userdir
sudo systemctl restart apache2
cd /home/vagrant/
mkdir public_html
cd public_html/
nano index.html
curl localhost
curl '
http://localhost/~vagrant
'
    
$ find -printf '%T+ %p\n'|sort|tail -5
        
/etc/apache2/mods-available/userdir.conf

'a2enmod userdir':
  
cmd.run
:
    - creates: /etc/apache2/mods-enabled/userdir.load

## Manuaali

sudo salt-call --local sys.state_doc
 
        

        


