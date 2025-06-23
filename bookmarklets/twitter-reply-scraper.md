# Twitter Replies Scraper

Example tweet with over 100 replies: https://x.com/DefenderOfBasic/status/1932423154086867451. 

Just paste this in the browser console:

```
// Auto-scroll and extract all Twitter replies without duplicates (EXCLUDES OP)
let extractedReplies = new Set();
let allReplies = [];
let lastScrollHeight = 0;
let noNewContentCount = 0;
let currentScrollPosition = 0;
let hitDiscoverMore = false;
let opUsername = null;

// Get the OP username from the first tweet
function getOPUsername() {
 const firstTweet = document.querySelector('[data-testid="tweet"]');
 if (firstTweet) {
   const userInfo = firstTweet.querySelector('[data-testid="User-Name"]') || firstTweet.querySelector('a[href*="/"]');
   if (userInfo) {
     const username = userInfo.textContent.trim().split('@')[0];
     console.log(`🎯 Detected OP: "${username}" - will exclude from results`);
     return username;
   }
 }
 console.log('⚠️ Could not detect OP username');
 return null;
}

// Simple hash function that works with Unicode
function simpleHash(str) {
 let hash = 0;
 for (let i = 0; i < str.length; i++) {
   const char = str.charCodeAt(i);
   hash = ((hash << 5) - hash) + char;
   hash = hash & hash;
 }
 return Math.abs(hash).toString();
}

function checkForDiscoverMore() {
 const discoverMoreIndicators = [
   'Discover more',
   'More posts',
   'Recommended',
   'You might like',
   'Similar posts'
 ];
 
 const pageText = document.body.textContent.toLowerCase();
 
 for (let indicator of discoverMoreIndicators) {
   if (pageText.includes(indicator.toLowerCase())) {
     const elements = document.querySelectorAll('*');
     for (let element of elements) {
       if (element.textContent && element.textContent.trim().toLowerCase().includes(indicator.toLowerCase())) {
         const rect = element.getBoundingClientRect();
         if (rect.top >= 0 && rect.top <= window.innerHeight) {
           console.log(`🛑 Found "${indicator}" section - stopping extraction here`);
           return true;
         }
       }
     }
   }
 }
 return false;
}

function extractNewReplies() {
 if (hitDiscoverMore) return 0;
 
 if (checkForDiscoverMore()) {
   hitDiscoverMore = true;
   return 0;
 }
 
 const tweets = document.querySelectorAll('[data-testid="tweet"]');
 let newCount = 0;
 
 tweets.forEach((tweet) => {
   if (hitDiscoverMore) return;
   
   const tweetText = tweet.querySelector('[data-testid="tweetText"]');
   const userInfo = tweet.querySelector('[data-testid="User-Name"]') || tweet.querySelector('a[href*="/"]');
   const isReply = tweet.querySelector('[data-testid="reply"]') || tweet.textContent.includes('Replying to');
   const tweetLink = tweet.querySelector('a[href*="/status/"]').href;

   if (tweetText && isReply) {
     let username = userInfo ? userInfo.textContent.trim().split('@')[0] : 'Unknown';
     
     // Skip if this is the OP
     if (opUsername && username === opUsername) {
       return;
     }
     
     const textHash = simpleHash(tweetText.textContent.trim());
     
     if (!extractedReplies.has(textHash)) {
       extractedReplies.add(textHash);
       
       let text = tweetText.textContent.trim();
       text = text.replace(/"/g, '""');
       
       allReplies.push({ username, text, url: tweetLink });
       newCount++;
     }
   }
 });
 
 return newCount;
}

function createCSV() {
 let csv = 'username,text,url\n';
 allReplies.forEach(reply => {
   csv += `"${reply.username}","${reply.text}","${reply.url}"\n`;
 });
 return csv;
}

function checkForCompletion() {
 if (hitDiscoverMore) {
   clearInterval(scrollInterval);
   clearInterval(extractInterval);
   
   const csv = createCSV();
   navigator.clipboard.writeText(csv).then(() => {
     console.log(`🎉 COMPLETE! Hit "Discover more" section. Extracted ${allReplies.length} unique replies (excluding OP) and copied to clipboard!`);
     console.log('CSV Preview:', csv.substring(0, 300) + '...');
   }).catch(err => {
     console.error('Failed to copy to clipboard:', err);
     console.log('Final CSV:', csv);
   });
   return;
 }
 
 const currentHeight = document.body.scrollHeight;
 
 if (currentHeight === lastScrollHeight) {
   noNewContentCount++;
 } else {
   noNewContentCount = 0;
   lastScrollHeight = currentHeight;
 }
 
 if (noNewContentCount >= 3) {
   clearInterval(scrollInterval);
   clearInterval(extractInterval);
   
   const csv = createCSV();
   navigator.clipboard.writeText(csv).then(() => {
     console.log(`🎉 COMPLETE! Extracted ${allReplies.length} unique replies (excluding OP) and copied to clipboard!`);
     console.log('CSV Preview:', csv.substring(0, 300) + '...');
   }).catch(err => {
     console.error('Failed to copy to clipboard:', err);
     console.log('Final CSV:', csv);
   });
 }
}

// Start the process
console.log('🚀 Starting auto-scroll and extraction...');

// Get OP username first
opUsername = getOPUsername();

// Scroll more gradually
const scrollInterval = setInterval(() => {
 if (hitDiscoverMore) return;
 
 currentScrollPosition += 300;
 window.scrollTo(0, currentScrollPosition);
 
 if (currentScrollPosition >= document.body.scrollHeight - window.innerHeight) {
   window.scrollTo(0, document.body.scrollHeight);
 }
}, 200);

// Extract new replies every 1 second
const extractInterval = setInterval(() => {
 const newReplies = extractNewReplies();
 if (newReplies > 0) {
   console.log(`📝 Found ${newReplies} new replies (Total: ${allReplies.length})`);
 }
 checkForCompletion();
}, 1000);

console.log('⏳ Scrolling and extracting... Will stop at "Discover more" or when complete.');

```
