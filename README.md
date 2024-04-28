# Charlie's CSDS 285 Portfolio

### Scripts for Java TAs
This semester, I was a TA for CSDS 132, so I came up with a few simple aliases to help with grading projects.

#### Bulk compile/delete
```zsh
alias compile="ls | grep '.*.java' > sources.txt; javac @sources.txt"
alias clean="rm *.class; rm *.java"
```
TLDR:
- Compile and delete CSDS 132 project files very quickly

The first two 132 projects don't have a lot of files, so I was fine with compiling and deleting these by hand after grading.  The third project, however, had on average 10 files to compile and sift through, so I wrote these aliases to compile all of the project files at once and to delete all of them at once.  This made it very easy to verify that programs could actually run, and it probably **saved me at least 20 min per grading session**.

![Compile/Clean demo](/images/grading.png)

#### Switch Java version
```zsh
alias j21="export JAVA_HOME=`/usr/libexec/java_home -v 21`; java -version"
alias j17="export JAVA_HOME=`/usr/libexec/java_home -v 17`; java -version"
alias j8="export JAVA_HOME=`/usr/libexec/java_home -v 1.8`; java -version"
```
TLDR:
- Switch Java versions for different classes

In addition to TA'ing, I was also taking CSDS 293, which required a different version of Java than 132.  This meant that I was constantly having to switch between Java versions for class work and TA work.  These aliases made switching between versions way faster.

![Switch Java version](/images/switch.png)

---

### Scrape New York Times Spelling Bee Comments
I am an avid fan of the NYT Spelling Bee game, which involves making as many words as possible from a set of 7 words.  There are other even more devoted Spelling Bee fanatics, and they like to write comments as clues to the words in the community tab.

TLDR:
- This script scrapes Spelling Bee comments to retrieve clues

#### Version 1
Initially, I was not aware of any way to make php process a dynamic webpage, so I couldn't actually get the comments since a side panel needed to be opened to generate the comments.  Instead, I had the script just grab the hints grid on the static webpage.

##### scraper.php
```php
error_reporting(0);

$url = "https://www.nytimes.com/2024/04/25/crosswords/spelling-bee-forum.html";

$doc = new DOMDocument();

$doc->loadHTMLFile($url);

$finder = new DOMXPath($doc);

$query = "//*[contains(@class, 'content') or contains(@class, 'table')]";

$hints = $finder->query($query);

echo "<div>";
foreach ($hints as $item) {
    echo $doc->saveHTML($item);
}

echo "</div>";
```

![Hints grid](images/grid.png)

<br/>

#### Version 2
I used ChatGPT to help me come up with a solution to retrieve the contents of the dynamic webpage.  It suggested I use a headless browser, such as Puppeteer for JavaScript.  I had ChatGPT write the script for spinning up the headless browser, and I tweaked the php script to call the .js file.

##### fetch_comments.js
```javascript
const puppeteer = require("puppeteer");

(async () => {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();
  await page.goto(
    "https://www.nytimes.com/2024/04/25/crosswords/spelling-bee-forum.html#commentsContainer"
  );

  await page.waitForSelector(".css-199z855");

  const comments = await page.evaluate(() => {
    const commentElements = document.querySelectorAll(
      ".css-199z855 .css-1ep7e7p"
    );
    const comments = [];
    commentElements.forEach((element) => {
      comments.push(element.textContent.trim());
    });
    return comments;
  });

  console.log(comments);

  await browser.close();
})();
```
##### scraper.php
```php
// Command to run JavaScript file
$command = 'node fetch_comments.js';

$output = shell_exec($command);

echo "<h2>Comments:</h2>";
echo "<ul>";
foreach (explode("\n", $output) as $comment) {
    $comment = preg_replace("/\'/i", "", trim($comment, '+'));
    $cleancomment = preg_replace("/\\\\n/i", "", $comment);
    echo "<li>$cleancomment</li>";
}
echo "</ul>";
```
![Spelling Bee Hints](images/hints.png)

---

### LeetCode Problem Tracker and Spaced Repetition Reminder
I wanted an easy way to keep track of completed LeetCode problems, as well as a way of reminding myself when to revisit questions to reinforce my learning.  

TLDR:

- Site to input completed problems and highlight problems a week later

This php site allows the user to input the title, link, and difficulty of a problem.  Once a week has passed, the site will highlight questions that need to be revisited.  The initial version was very barebones and ugly, so I had ChatGPT help with all of the styling and creating the HTML table.  I also had wanted to see if ChatGPT could seamlessly add in the remove button into my code, so that was also added in a later revision.

##### LeetCode Tracker
~~~php
<?php
error_reporting(0);

function track_problem($problem, $link, $difficulty) {
    $timestamp = date("Y-m-d H:i:s");
    $line = "$timestamp, $problem, $link, $difficulty\n";
    file_put_contents('leetcode_tracker.txt', $line, FILE_APPEND);
}

function remove_problem($index) {
    $file = 'leetcode_tracker.txt';
    $lines = file($file);
    if (isset($lines[$index])) {
        unset($lines[$index]);
        file_put_contents($file, implode('', $lines));
    }
}

function display_problems() {
    $file = 'leetcode_tracker.txt';
    $lines = file($file);
    $current_date = strtotime(date("Y-m-d"));

    echo "<h2 style='text-align: center; margin-top: 20px;'>LeetCode Problems Tracker</h2>";
    echo "<div style='margin: 0 auto; width: 70%;'>";
    echo "<table border='1' cellpadding='10' style='border-collapse: collapse; width: 100%;'>";
    echo "<tr><th style='background-color: #f2f2f2; text-align: center; padding: 10px;'>Date</th><th style='background-color: #f2f2f2; text-align: center; padding: 10px;'>Problem</th><th style='background-color: #f2f2f2; text-align: center; padding: 10px;'>Difficulty</th><th style='background-color: #f2f2f2; text-align: center; padding: 10px;'>Actions</th></tr>";

    foreach ($lines as $index => $line) {
        $data = explode(', ', $line);
        $date = strtotime($data[0]);
        $problem = $data[1];
        $link = $data[2];
        $difficulty = $data[3];

        // Check if a week has passed since solving the problem
        $highlightClass = ($current_date - $date) >= 604800 ? 'highlight' : '';

        echo "<tr>";
        echo "<td style='text-align: center; padding: 10px;'>$data[0]</td>";
        echo "<td style='text-align: center; padding: 10px;'><a href='$link' target='_blank'><span class='$highlightClass'>$problem</span></a></td>";
        echo "<td style='text-align: center; padding: 10px;'>$difficulty</td>";
        echo "<td style='text-align: center; padding: 10px;'><form method='post'><input type='hidden' name='index' value='$index'><input type='submit' name='remove' value='Remove'></form></td>";
        echo "</tr>";
    }

    echo "</table>";
    echo "</div>";
}

if ($_SERVER["REQUEST_METHOD"] == "POST" && isset($_POST['remove'])) {
    $index = $_POST['index'];
    remove_problem($index);
}

if ($_SERVER["REQUEST_METHOD"] == "POST" && isset($_POST['problem'])) {
    $problem = $_POST['problem'];
    $link = $_POST['link'];
    $difficulty = $_POST['difficulty'];

    track_problem($problem, $link, $difficulty);
}
?>

<!DOCTYPE html>
<html>
<head>
    <title>LeetCode Tracker</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f8f8f8;
            margin: 0;
            padding: 0;
        }
        h2, h3 {
            text-align: center;
        }
        form {
            margin: 20px auto;
            text-align: center;
            width: 70%;
        }
        input[type=text], select {
            width: 100%;
            padding: 10px;
            margin: 5px 0;
            display: block;
            border: 1px solid #ccc;
            border-radius: 4px;
            box-sizing: border-box;
        }
        input[type=submit] {
            width: 100%;
            background-color: #4CAF50;
            color: white;
            padding: 14px 20px;
            margin: 8px 0;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        input[type=submit]:hover {
            background-color: #45a049;
        }
        table {
            border-collapse: collapse;
            width: 100%;
            margin-top: 20px;
        }
        th, td {
            border: 1px solid #ddd;
            padding: 8px;
            text-align: left;
        }
        th {
            background-color: #f2f2f2;
        }
        tr:nth-child(even) {
            background-color: #f2f2f2;
        }
        .highlight {
            background-color: yellow;
        }
    </style>
</head>
<body>
    <?php display_problems(); ?>
    <h3 style="text-align: center;">Add a New Problem</h3>
    <form method="post" action="<?php echo htmlspecialchars($_SERVER["PHP_SELF"]);?>" style="text-align: center;">
        <label for="problem">Problem:</label>
        <input type="text" id="problem" name="problem"><br/>
        <label for="link">Link:</label>
        <input type="text" id="link" name="link"><br/>
        <label for="difficulty">Difficulty:</label>
        <select id="difficulty" name="difficulty">
            <option value="Easy">Easy</option>
            <option value="Medium">Medium</option>
            <option value="Hard">Hard</option>
        </select><br/>
        <input type="submit" value="Submit">
    </form>
</body>
</html>
~~~


![LeetCode Tracker](images/leetcodetracker.png)

