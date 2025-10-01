---
layout: default
title: Arşiv
---

<style>
  .archive-container {
    margin: 20px 0;
  }

  .archive-months {
    display: flex;
    flex-direction: column;
    gap: 40px;
  }

  .month-section {
    margin-bottom: 30px;
  }

  .month-header {
    font-size: 24px;
    font-weight: 700;
    color: #dc2626;
    margin-bottom: 20px;
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
    height: 90px;
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

  .day-cell.has-post:hover {
    background: linear-gradient(135deg, #fee2e2 0%, #fecaca 100%);
    box-shadow: 0 4px 8px rgba(220, 38, 38, 0.2);
    transform: translateY(-2px);
  }

  .day-cell.empty {
    background: transparent;
    border: none;
    height: 90px;
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

<div class="archive-container">
  <h1>Arşiv</h1>
  <div id="archive-content"></div>
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
      post.year = parseInt(year);
      post.month = parseInt(month);
    }
  });

  // Filter out posts without valid dates
  const validPosts = posts.filter(p => p.date);

  // Sort posts by date (newest first)
  validPosts.sort((a, b) => b.date - a.date);

  if (validPosts.length === 0) return;

  // Group posts by month
  const postsByMonth = {};
  validPosts.forEach(post => {
    const monthKey = `${post.year}-${String(post.month).padStart(2, '0')}`;
    if (!postsByMonth[monthKey]) {
      postsByMonth[monthKey] = [];
    }
    postsByMonth[monthKey].push(post);
  });

  // Get all month keys sorted (newest first)
  const monthKeys = Object.keys(postsByMonth).sort().reverse();

  // Turkish month names
  const monthNames = [
    'Ocak', 'Şubat', 'Mart', 'Nisan', 'Mayıs', 'Haziran',
    'Temmuz', 'Ağustos', 'Eylül', 'Ekim', 'Kasım', 'Aralık'
  ];

  // Build the archive HTML
  let archiveHtml = '<div class="archive-months">';

  // Add calendar header once
  archiveHtml += `
    <div class="calendar-header">
      <div class="day-name">Pzt</div>
      <div class="day-name">Sal</div>
      <div class="day-name">Çar</div>
      <div class="day-name">Per</div>
      <div class="day-name">Cum</div>
    </div>
  `;

  monthKeys.forEach(monthKey => {
    const [year, month] = monthKey.split('-');
    const monthName = monthNames[parseInt(month) - 1];
    const monthPosts = postsByMonth[monthKey];

    archiveHtml += `
      <div class="month-section">
        <div class="month-header">${monthName} ${year}</div>
    `;

    // Create a map of posts by date for this month
    const postsByDate = {};
    monthPosts.forEach(post => {
      const dateKey = `${post.year}-${String(post.month).padStart(2, '0')}-${String(post.date.getDate()).padStart(2, '0')}`;
      postsByDate[dateKey] = post;
    });

    // Find the first and last day of the month
    const firstDay = new Date(parseInt(year), parseInt(month) - 1, 1);
    const lastDay = new Date(parseInt(year), parseInt(month), 0);

    // Find the Monday of the week containing the first day
    let currentDate = new Date(firstDay);
    const dayOfWeek = currentDate.getDay();
    const daysToMonday = dayOfWeek === 0 ? 6 : dayOfWeek - 1;
    currentDate.setDate(currentDate.getDate() - daysToMonday);

    // Generate calendar weeks
    while (currentDate <= lastDay) {
      let weekHtml = '';
      let hasPostsInWeek = false;

      // Process week (Monday to Friday)
      for (let i = 0; i < 5; i++) {
        const checkDate = new Date(currentDate);
        checkDate.setDate(checkDate.getDate() + i);

        const checkYear = checkDate.getFullYear();
        const checkMonth = checkDate.getMonth() + 1;
        const checkDay = checkDate.getDate();
        const dateKey = `${checkYear}-${String(checkMonth).padStart(2, '0')}-${String(checkDay).padStart(2, '0')}`;

        const post = postsByDate[dateKey];

        // Only show cells for days in the current month
        if (checkMonth === parseInt(month) && checkYear === parseInt(year)) {
          if (post) {
            weekHtml += `
              <div class="day-cell has-post">
                <div class="day-content">
                  <a href="${post.url}">${post.title}</a>
                </div>
              </div>`;
            hasPostsInWeek = true;
          } else {
            weekHtml += '<div class="day-cell empty"></div>';
          }
        } else {
          weekHtml += '<div class="day-cell empty"></div>';
        }
      }

      if (hasPostsInWeek) {
        archiveHtml += '<div class="calendar-grid">' + weekHtml + '</div>';
      }

      // Move to next week (Monday)
      currentDate.setDate(currentDate.getDate() + 7);
    }

    archiveHtml += `
      </div>
    `;
  });

  archiveHtml += '</div>';

  document.getElementById('archive-content').innerHTML = archiveHtml;
});
</script>
