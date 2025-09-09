---
layout: default
title: Anasayfa
---

<style>
  .calendar-container {
    margin: 20px 0;
  }
  
  .calendar-header {
    display: grid;
    grid-template-columns: repeat(5, 1fr);
    gap: 8px;
    margin-bottom: 20px;
    padding: 12px;
    background: linear-gradient(135deg, #dc2626 0%, #991b1b 100%);
    border-radius: 8px;
  }
  
  .calendar-header .day-name {
    text-align: center;
    font-weight: 600;
    font-size: 14px;
    color: white;
    text-transform: uppercase;
    letter-spacing: 0.5px;
  }
  
  .calendar-grid {
    display: grid;
    grid-template-columns: repeat(5, 1fr);
    gap: 8px;
    margin-bottom: 15px;
  }
  
  .day-cell {
    min-height: 90px;
    border: 2px solid #e1e4e8;
    border-radius: 8px;
    padding: 12px;
    background: #fafbfc;
    transition: all 0.2s ease;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  
  .day-cell.has-post {
    background: white;
    border-color: #dc2626;
    cursor: pointer;
    position: relative;
    padding: 0;
  }
  
  .day-cell.has-post.most-recent::before {
    content: '';
    position: absolute;
    top: 8px;
    right: 8px;
    width: 8px;
    height: 8px;
    background: #000000;
    border-radius: 50%;
  }
  
  .day-cell.has-post:hover {
    background: linear-gradient(135deg, #fee2e2 0%, #fecaca 100%);
    box-shadow: 0 4px 8px rgba(220, 38, 38, 0.2);
    transform: translateY(-2px);
  }
  
  .day-cell.empty {
    background: transparent;
    border: none;
  }
  
  .day-content {
    width: 100%;
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  
  .day-content a {
    color: #dc2626;
    text-decoration: none;
    font-size: 16px;
    font-weight: 600;
    display: flex;
    align-items: center;
    justify-content: center;
    width: 100%;
    height: 100%;
    padding: 12px;
    text-align: center;
  }
  
  .day-content a:hover {
    text-decoration: none;
  }
  
  @media (max-width: 768px) {
    .calendar-grid {
      grid-template-columns: 1fr;
    }
    .calendar-header {
      display: none;
    }
    .day-cell {
      min-height: 60px;
    }
  }
</style>

<div class="calendar-container">
  <div class="calendar-header">
    <div class="day-name">Pzt</div>
    <div class="day-name">Sal</div>
    <div class="day-name">Çar</div>
    <div class="day-name">Per</div>
    <div class="day-name">Cum</div>
  </div>
  
  <div id="calendar-content"></div>
</div>

<script>
document.addEventListener('DOMContentLoaded', function() {
  const posts = [
    {% for post in site.posts %}
    {
      url: '{{ post.url }}',
      title: '{{ post.title | escape }}',
      dateFromUrl: '{{ post.url }}'.match(/\/(\d{4})\/(\d{2})\/(\d{2})\//),
    }{% unless forloop.last %},{% endunless %}
    {% endfor %}
  ];
  
  // Process posts to extract proper dates from URLs
  posts.forEach(post => {
    if (post.dateFromUrl) {
      const [, year, month, day] = post.dateFromUrl;
      post.date = new Date(parseInt(year), parseInt(month) - 1, parseInt(day));
    }
  });
  
  // Filter out posts without valid dates
  const validPosts = posts.filter(p => p.date);
  
  // Group posts by date string
  const postsByDate = {};
  validPosts.forEach(post => {
    const year = post.date.getFullYear();
    const month = String(post.date.getMonth() + 1).padStart(2, '0');
    const day = String(post.date.getDate()).padStart(2, '0');
    const dateKey = `${year}-${month}-${day}`;
    postsByDate[dateKey] = post;
  });
  
  // Sort posts by date (newest first)
  validPosts.sort((a, b) => b.date - a.date);
  
  if (validPosts.length === 0) return;
  
  // Find the most recent post date for marking
  const mostRecentPostDate = validPosts[0].date;
  const mostRecentKey = `${mostRecentPostDate.getFullYear()}-${String(mostRecentPostDate.getMonth() + 1).padStart(2, '0')}-${String(mostRecentPostDate.getDate()).padStart(2, '0')}`;
  
  const startDate = new Date(validPosts[validPosts.length - 1].date);
  const endDate = new Date(validPosts[0].date);
  
  // Add a few days buffer to make sure we capture all weeks
  startDate.setDate(startDate.getDate() - 7);
  endDate.setDate(endDate.getDate() + 7);
  
  let currentDate = new Date(startDate);
  const calendarContent = document.getElementById('calendar-content');
  const allWeeks = [];
  
  // Find the Monday of the week containing startDate
  const dayOfWeek = currentDate.getDay();
  const daysToMonday = dayOfWeek === 0 ? 6 : dayOfWeek - 1;
  currentDate.setDate(currentDate.getDate() - daysToMonday);
  
  while (currentDate <= endDate) {
    let weekHtml = '';
    let hasPostsInWeek = false;
    
    // Process week (Monday to Friday)
    for (let i = 0; i < 5; i++) {
      const checkDate = new Date(currentDate);
      checkDate.setDate(checkDate.getDate() + i);
      
      const checkYear = checkDate.getFullYear();
      const checkMonth = String(checkDate.getMonth() + 1).padStart(2, '0');
      const checkDay = String(checkDate.getDate()).padStart(2, '0');
      const dateKey = `${checkYear}-${checkMonth}-${checkDay}`;
      
      const post = postsByDate[dateKey];
      
      if (post) {
        const isMostRecent = dateKey === mostRecentKey;
        weekHtml += `
          <div class="day-cell has-post${isMostRecent ? ' most-recent' : ''}">
            <div class="day-content">
              <a href="${post.url}">${post.title}</a>
            </div>
          </div>`;
        hasPostsInWeek = true;
      } else {
        weekHtml += '<div class="day-cell"></div>';
      }
    }
    
    if (hasPostsInWeek) {
      allWeeks.push('<div class="calendar-grid">' + weekHtml + '</div>');
    }
    
    // Move to next week (Monday)
    currentDate.setDate(currentDate.getDate() + 7);
  }
  
  // Reverse the weeks array to show most recent first
  allWeeks.reverse();
  
  // Add all weeks to the calendar
  calendarContent.innerHTML = allWeeks.join('');
});
</script>