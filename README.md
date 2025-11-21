# pharma-ai-dashboard[requirements.txt](https://github.com/user-attachments/files/23672687/requirements.txt)
streamlit==1.28.0
feedparser==6.0.10
google-generativeai==0.3.1
[app.py](https://github.com/user-attachments/files/23672691/app.py)
import streamlit as st
import feedparser
from datetime import datetime
import google.generativeai as genai

# Configure the page
st.set_page_config(page_title="Pharma AI Intelligence Dashboard", layout="wide")

# HARD-CODED API KEY
GEMINI_API_KEY = "AIzaSyAcJB7RddhqOfxvUO6W59dmaxHT-n0yyyo"

# Configure Gemini
genai.configure(api_key=GEMINI_API_KEY)

# RSS FEEDS - Pharma Marketing & AI focused
RSS_FEEDS = {
    "MM&M": "https://www.mmm-online.com/feed/",
    "PM360": "https://www.pm360online.com/feed/",
    "Pharmaphorum": "https://pharmaphorum.com/rssfeed/news",
    "PharmaVOICE": "https://www.pharmavoice.com/feeds/news",
    "AllazoHealth AI": "https://allazohealth.com/resources/feed/",
    "Healthcare IT News AI": "https://www.healthcareitnews.com/taxonomy/term/1577/feed",
    "Fierce Pharma": "https://www.fiercepharma.com/rss",
    "Pharma Commerce": "https://pharmaceuticalcommerce.com/rss/"
}

def fetch_feed(url, max_items=5):
    try:
        feed = feedparser.parse(url)
        return feed.entries[:max_items]
    except Exception as e:
        st.error(f"Error fetching feed: {e}")
        return []

def analyze_innovation_and_ai(article_text):
    try:
        model = genai.GenerativeModel('gemini-1.5-flash')
        
        prompt = f"""You are analyzing pharmaceutical marketing and communications content.

Article: {article_text}

Provide analysis in the following format:

**Innovation Score (1-5):** [Rate how innovative this company/initiative is]

**Generative AI Usage:** [Explicitly state if they mention using generative AI, and if so, HOW they are using it. If no mention, say "No generative AI mentioned"]

**Key Takeaway:** [One sentence summary of the most important point]

Keep your response concise and structured exactly as shown above."""
        
        response = model.generate_content(prompt)
        return response.text
    except Exception as e:
        return f"Error: {str(e)}"

def generate_whitespace_analysis(all_articles):
    try:
        model = genai.GenerativeModel('gemini-1.5-flash')
        
        prompt = f"""You are a pharmaceutical marketing strategist analyzing competitive intelligence.

Based on these pharma marketing and communications articles:
{all_articles}

Provide a strategic analysis:

## Current Market Landscape
[Summarize what companies ARE doing with AI and innovation]

## Generative AI Adoption Patterns
[Specific examples of how companies are discussing or using generative AI]

## White Space Opportunities for a Pharma Marketing Agency
[3-5 specific AI services/capabilities that are NOT being widely offered yet, based on gaps in the market]

## Positioning Recommendations
[How should a pharmaceutical marketing agency position their AI offering to stand out?]

Be specific and actionable. Focus on what's MISSING in the market."""
        
        response = model.generate_content(prompt)
        return response.text
    except Exception as e:
        return f"Error: {str(e)}"

st.title("Pharma AI Competitive Intelligence Dashboard")
st.markdown("*Tracking innovation and generative AI adoption in pharmaceutical marketing*")
st.markdown("---")

st.sidebar.header("Dashboard Settings")

selected_feeds = st.sidebar.multiselect(
    "Select News Sources:",
    options=list(RSS_FEEDS.keys()),
    default=list(RSS_FEEDS.keys())
)

num_articles = st.sidebar.slider("Articles per source:", 3, 8, 5)

show_article_analysis = st.sidebar.checkbox("Show Individual Article Analysis", value=True)
show_whitespace = st.sidebar.checkbox("Generate White Space Analysis", value=True)

if st.sidebar.button("Refresh Dashboard"):
    st.rerun()

all_articles_text = ""
article_count = 0

for feed_name in selected_feeds:
    st.header(f"{feed_name}")
    
    with st.spinner(f"Loading {feed_name}..."):
        entries = fetch_feed(RSS_FEEDS[feed_name], num_articles)
    
    if entries:
        for entry in entries:
            article_count += 1
            
            article_text = f"Source: {feed_name}\nTitle: {entry.title}\n"
            if hasattr(entry, 'summary'):
                article_text += f"Summary: {entry.summary[:400]}\n\n"
            
            all_articles_text += article_text
            
            with st.expander(f"**{entry.title}**"):
                if hasattr(entry, 'published'):
                    st.caption(f"Published: {entry.published}")
                
                if hasattr(entry, 'summary'):
                    st.write(entry.summary[:300] + "...")
                
                if show_article_analysis:
                    with st.spinner("Analyzing innovation & AI usage..."):
                        analysis = analyze_innovation_and_ai(article_text)
                        st.markdown("---")
                        st.markdown("### AI Analysis")
                        st.info(analysis)
                
                if hasattr(entry, 'link'):
                    st.markdown(f"[Read full article]({entry.link})")
    else:
        st.warning(f"No articles found for {feed_name}")
    
    st.markdown("---")

if show_whitespace and all_articles_text:
    st.header("Strategic White Space Analysis")
    st.markdown("*AI-powered competitive intelligence for positioning your agency's offerings*")
    
    with st.spinner("Analyzing market gaps and opportunities..."):
        whitespace = generate_whitespace_analysis(all_articles_text[:15000])
        
        st.success(whitespace)

st.sidebar.markdown("---")
st.sidebar.metric("Articles Analyzed", article_count)
st.sidebar.info(f"Last updated: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
