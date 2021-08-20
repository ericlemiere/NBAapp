# NBAapp
 
During a 2-week sprint, I was tasked with making a web application that scraped certain data from another webpage and displayed that data in a useable way. Because I am heavily involved in fantasy basketball leagues, I chose to get defensive stats for NBA players and add them up to see how many defensive fantasy points each player scored for the 2020-2021 season. To make this application happen, I used Django and incorporated HTML, CSS, Javascript, Python, and SQL to make a nice looking website with the ability to save players to different databases, as well as edit and delete from the databases. 
Here are some code snippets and screenshots of the web app:


## Home Page

<img width="1435" alt="nba home" src="https://user-images.githubusercontent.com/83970546/127374260-e485eea8-48fb-427c-95d2-1ef464eef173.png">


This is the method that gets the data from a website and saves that data into my database:

    def b_ref(request):
        # URL page to scrape
        brURL = "https://www.basketball-reference.com/leagues/NBA_2021_totals.html"
        page = requests.get(brURL)
        soup = BeautifulSoup(page.content, 'html.parser')

        rows = soup.findAll('tr')[1:]

        # Extract table data (td) from the rows and put into list
        allStats = [[td.getText() for td in rows[i].findAll('td')]
                    for i in range(len(rows))]

        # connect to database, and clear it so that it's empty
        conn = sqlite3.connect('db.sqlite3')
        with conn:
            cur = conn.cursor()
            cur.execute("DELETE FROM NBAapp_PlayerDb")

        player_dict = []
        check_name = ''
        pk = 1
        for row in allStats:
            if len(row) > 0:  # check to see if list is empty
                playerName = row[0]
                defRebs = int(row[21])
                steals = int(row[24])
                blocks = int(row[25])
                total_def_points = defRebs + (steals * 3) + (blocks * 3)
                if total_def_points >= 200:
                    if playerName != check_name:
                        myStats = {'playerName': playerName,
                                   'defRebs': defRebs,
                                   'steals': steals,
                                   'blocks': blocks,
                                   'total_def_points': total_def_points,
                                   'pk': pk}
                        player_dict.append(myStats)

                        conn = sqlite3.connect('db.sqlite3')
                        with conn:
                            cur = conn.cursor()
                            cur.execute('INSERT INTO NBAapp_PlayerDb VALUES (?,?,?,?,?,?)',
                                        [pk, playerName, defRebs, steals, blocks, total_def_points])
                            conn.commit()
                        pk += 1  # this gets set as the primary key
                    check_name = playerName
        context = {'player_dict': player_dict}
        return render(request, 'nba-home.html', context)



Clicking on a player row brings up a hidden div with that player's info:

<img width="1433" alt="nba home 2" src="https://user-images.githubusercontent.com/83970546/127377105-6845f241-4b68-495c-85a9-ecdf8908a542.png">


## Column Sort Function using Javascript
By clicking on the column headers on the home page, a user can sort those columns in order from highest to lowest. Here is the javascript code for that:
```
function sortTable(column) {
  var table, rows, switching, i, x, y, shouldSwitch;
  table = document.getElementById("myTable");
  switching = true;
  while (switching) {
    switching = false;
    rows = table.rows;
    // Loop through all table rows (except the
    // first, which contains table headers):
    for (i = 1; i < (rows.length - 1); i++) {
      shouldSwitch = false;
      // Get the two elements you want to compare,
      // one from current row and one from the next:
      xString = rows[i].getElementsByTagName("TD")[column].innerHTML;
      yString = rows[i + 1].getElementsByTagName("TD")[column].innerHTML;
      x = parseInt(xString);
      y = parseInt(yString);
      //check if the two rows should switch place:
      if (x < y) {
        //if so, mark as a switch and break the loop:
        shouldSwitch = true;
        break;
      }
    }
    if (shouldSwitch) {
      // If a switch has been marked, make the switch
      // and mark that a switch has been done:
      rows[i].parentNode.insertBefore(rows[i + 1], rows[i]);
      switching = true;
    }
  }
}
```


## Favorites
As you can see on the home page, the user has the ability to save a player to a favorites database, as seen here:

<img width="1434" alt="nba app favorites" src="https://user-images.githubusercontent.com/83970546/127375874-fb1dc34a-82c9-4a1e-9519-696ba49a2f10.png">


When a player is removed from the favorites database, the user sees the following page for 2 seconds and is redirected back to the favorites page:

<img width="1424" alt="nba app removed" src="https://user-images.githubusercontent.com/83970546/127375032-3834c6ba-5b6d-47b7-b1a5-7d35e07c82fd.png">


## Things Learned
- How to work on a project with other developers using Microsoft Azure
- How to create a fully-functioning web app using Django
- Furthering my knowledge on HTML, CSS, Javascript, Python, and SQL
