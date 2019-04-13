# CTI Crawlers 


This is a orchestration of crawlers, with more complex traversal definitions ie., the engine starts with 
a blog_list crawler and it will go through two traversals - one is pagination and the other is gathering the 
details of the extracted list of blogs. 

```yaml

cti_id: scrapinghub_blogs
init_crawler:
  start_urls:
  - "https://blog.scrapinghub.com"
  crawler_id: blog_list
crawlers:
- crawler_id: blog_list
  parsers:
  - parser_type: HTMLMetaTagExtractor
    parser_id: meta_tags
  - parser_type: ParagraphExtractor
    parser_id: paragraphs
  - parser_type: CustomContentExtractor
    parser_id: blog_list_parser
    data_selectors:
    - selector_id: blogs
      selector: ".post-listing .post-item"
      selector_attribute: element
      multiple: true
      child_selectors:
      - selector_id: url
        selector: ".post-header h2 a"
        selector_type: css
        selector_attribute: href
        multiple: false
      - selector_id: title
        selector: ".post-header h2 a"
        selector_type: css
        selector_attribute: text
        multiple: false
      - selector_id: content
        selector: ".post-content"
        selector_type: css
        selector_attribute: html
        multiple: false
  traversals:
  - traversal_type: pagination
    pagination:
      selector: ".next-posts-link"
      selector_type: css
      max_pages: 1
    next_crawler_id: blog_list
  - traversal_type: link_from_field
    link_from_field:
      parser_id: blog_list_parser
      selector_id: url
    next_crawler_id: blog_detail
- crawler_id: blog_detail
  parsers:
  - parser_type: CustomContentExtractor
    parser_id: blog_detail
    data_selectors:
    - selector_id: blog_detail
      selector: ".blog-section"
      selector_attribute: element
      multiple: false
      child_selectors:
      - selector_id: title
        selector: h1 span
        selector_type: css
        selector_attribute: text
        multiple: false
      - selector_id: published_at
        selector: ".date a"
        selector_type: css
        selector_attribute: text
        multiple: false
      - selector_id: author
        selector: ".author a"
        selector_type: css
        selector_attribute: text
        multiple: false
      - selector_id: html_content
        selector: ".post-body"
        selector_type: css
        selector_attribute: html
        multiple: false
transformations:
- transformation_id: default
  transformation_fn: transformation_fn
indexes:
- index_id: primary_db
  transformation_id: default
  connection_uri: mongodb://127.0.0.1/crawlers_data_index
  collection_name: blog_list
  unique_key: url
callbacks:
- callback_id: default
  index_id: default
  url: http://localhost/api/callback
  request_type: POST
  payload: {}
  headers:
    X-TOKEN: abc123456789
settings:
  allowed_domains:
  - "blog.scrapinghub.com"
  download_delay: 0
context:
  author: https://github.com/rrmerugu
  description: Crawler that scrapes invanalabs xyz

```