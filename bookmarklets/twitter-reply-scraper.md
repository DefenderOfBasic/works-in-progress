# Twitter Replies Scraper

Example tweet with over 100 replies: https://x.com/DefenderOfBasic/status/1932423154086867451. 

```
javascript:(function() { 
///////////////////////// SCRIPT BEGIN

// Auto-scroll and extract all Twitter replies without duplicates
let extractedReplies = new Set(); // Use Set to avoid duplicates
let allReplies = [];
let lastScrollHeight = 0;
let noNewContentCount = 0;

function extractNewReplies() {
  const tweets = document.querySelectorAll('[data-testid="tweet"]');
  let newCount = 0;
  
  tweets.forEach((tweet) => {
    const tweetText = tweet.querySelector('[data-testid="tweetText"]');
    const userInfo = tweet.querySelector('[data-testid="User-Name"]') || tweet.querySelector('a[href*="/"]');
    const isReply = tweet.querySelector('[data-testid="reply"]') || tweet.textContent.includes('Replying to');
    
    if (tweetText && isReply) {
      // Create unique hash from text content
      const textHash = btoa(tweetText.textContent.trim()).replace(/[^a-zA-Z0-9]/g, '');
      
      if (!extractedReplies.has(textHash)) {
        extractedReplies.add(textHash);
        
        let username = userInfo ? userInfo.textContent.trim() : 'Unknown';
        username = username.split('@')[0];
        
        let text = tweetText.textContent.trim();
        text = text.replace(/"/g, '""');
        
        allReplies.push({ username, text });
        newCount++;
      }
    }
  });
  
  return newCount;
}

function createCSV() {
  let csv = 'username,text\n';
  allReplies.forEach(reply => {
    csv += `"${reply.username}","${reply.text}"\n`;
  });
  return csv;
}

function checkForCompletion() {
  const currentHeight = document.body.scrollHeight;
  
  if (currentHeight === lastScrollHeight) {
    noNewContentCount++;
  } else {
    noNewContentCount = 0;
    lastScrollHeight = currentHeight;
  }
  
  // If no new content for 3 seconds (30 checks), we're probably done
  if (noNewContentCount >= 30) {
    clearInterval(scrollInterval);
    clearInterval(extractInterval);
    
    const csv = createCSV();
    navigator.clipboard.writeText(csv).then(() => {
      console.log(`🎉 COMPLETE! Extracted ${allReplies.length} unique replies and copied to clipboard!`);
      console.log('CSV Preview:', csv.substring(0, 300) + '...');
    }).catch(err => {
      console.error('Failed to copy to clipboard:', err);
      console.log('Final CSV:', csv);
    });
  }
}

// Start the process
console.log('🚀 Starting auto-scroll and extraction...');

// Scroll every 10ms
const scrollInterval = setInterval(() => {
  window.scrollTo(0, document.body.scrollHeight);
}, 10);

// Extract new replies every 500ms and check for completion
const extractInterval = setInterval(() => {
  const newReplies = extractNewReplies();
  if (newReplies > 0) {
    console.log(`📝 Found ${newReplies} new replies (Total: ${allReplies.length})`);
  }
  checkForCompletion();
}, 500);

console.log('⏳ Scrolling and extracting... Will auto-complete when done.');

///////////////// SCRIPT END
})();
```
