# Custom Subscription Templates

This directory contains example custom HTML templates for the subscription page feature. These templates can be used to customize how subscribers see their subscription information when visiting `/sub/:id` in a web browser.

## How to Use

1. Copy one of the templates to `/etc/x-ui/templates/sub.html`:

   ```bash
   # Copy the minimal (feature-rich) template
   sudo cp sub-minimal.html /etc/x-ui/templates/sub.html
   
   # Or copy the simple template
   sudo cp sub-simple.html /etc/x-ui/templates/sub.html
   
   # Or copy the dark theme template
   sudo cp sub-dark.html /etc/x-ui/templates/sub.html
   ```

2. Visit your subscription page in a browser: `http://your-server/sub/your-subscription-id`

3. The custom template will be automatically loaded and rendered.

## Available Templates

### 1. **sub-minimal.html** (Recommended)
- Modern, colorful gradient design
- Feature-rich with copy/download buttons
- Responsive grid layout
- Interactive proxy list
- Best for user-friendly experience

### 2. **sub-simple.html** (Balanced)
- Clean, professional look
- Compact stats display
- Simple and fast loading
- Good balance between features and simplicity
- Light/dark modern design

### 3. **sub-dark.html** (Minimal)
- Ultra-minimalist dark theme
- Small file size, fast loading
- Monospace font for code
- Perfect for technical users
- Low bandwidth friendly

## Template Variables

All templates have access to these variables:

| Variable | Type | Example | Description |
|----------|------|---------|-------------|
| `{{.sId}}` | string | `abc123def` | Subscription ID |
| `{{.download}}` | string | `512 MB` | Formatted download traffic |
| `{{.upload}}` | string | `1.5 GB` | Formatted upload traffic |
| `{{.total}}` | string | `10 GB` | Formatted total quota |
| `{{.used}}` | string | `2 GB` | Formatted used traffic |
| `{{.remained}}` | string | `8 GB` | Formatted remaining quota |
| `{{.expire}}` | int64 | `1735689600` | Unix timestamp (seconds) |
| `{{.lastOnline}}` | int64 | `1725689600` | Last online timestamp |
| `{{.downloadByte}}` | int64 | `536870912` | Download in bytes |
| `{{.uploadByte}}` | int64 | `1610612736` | Upload in bytes |
| `{{.totalByte}}` | int64 | `10737418240` | Total quota in bytes |
| `{{.subUrl}}` | string | `vmess://...` | Subscription URL (all links) |
| `{{.subJsonUrl}}` | string | `http://...` | JSON subscription URL |
| `{{.subClashUrl}}` | string | `http://...` | Clash subscription URL |
| `{{.result}}` | []string | `[...]` | Array of individual proxy links |
| `{{.host}}` | string | `example.com` | Server hostname |
| `{{.cur_ver}}` | string | `2.0.0` | Panel version |
| `{{.datepicker}}` | string | `gregorian` | Date format type |

## Creating Your Own Template

You can create a custom template using Go's `text/template` syntax:

```html
<!DOCTYPE html>
<html>
<head>
    <title>My Subscription</title>
</head>
<body>
    <h1>Your Subscription</h1>
    <p>ID: <strong>{{.sId}}</strong></p>
    <p>Usage: <strong>{{.used}}</strong> / {{.total}}</p>
    
    <h2>Proxy Links</h2>
    <ul>
    {{range .result}}
        <li><code>{{.}}</code></li>
    {{end}}
    </ul>
</body>
</html>
```

## Syntax Reference

### Displaying Variables
```html
<!-- Simple variable -->
ID: {{.sId}}

<!-- Conditional - show if value exists -->
{{if .remained}}Remaining: {{.remained}}{{end}}

<!-- Conditional - check if > 0 -->
{{if gt .totalByte 0}}Limited quota{{end}}

<!-- Conditional - check if = 0 -->
{{if eq .expire 0}}Never expires{{end}}

<!-- Loops -->
{{range .result}}
    <p>{{.}}</p>
{{end}}

<!-- Loops with counter -->
{{range $idx, $item := .result}}
    <p>{{add $idx 1}}. {{$item}}</p>
{{end}}
```

## Troubleshooting

### Template Not Loading
1. Check that file is at `/etc/x-ui/templates/sub.html` (exact path)
2. Check permissions: `sudo chmod 644 /etc/x-ui/templates/sub.html`
3. Check panel logs for parsing errors

### Invalid Syntax Error
- Ensure Go template syntax is correct (double curly braces `{{` not single `{`)
- Common mistake: using `{{if}}` instead of `{{if condition}}`
- Always close conditionals: `{{if}} ... {{end}}`

### Fallback to Built-in
If your template has errors, the panel will:
1. Log a warning message
2. Fall back to the built-in template
3. Render normally for the user

Check logs for details: `tail -f /var/log/x-ui/3xipl.log`

## Testing Your Template

### Before Deploying
1. Copy template to `/etc/x-ui/templates/sub.html`
2. Visit `http://your-server/sub/your-id` in browser
3. Check browser console (F12) for JavaScript errors
4. Check panel logs for template errors

### Debugging
Add temporary debug info to your template:
```html
<!-- Display raw data -->
<pre>
Download Byte: {{.downloadByte}}
Upload Byte: {{.uploadByte}}
Total Byte: {{.totalByte}}
Expire: {{.expire}}
Result Count: {{len .result}}
</pre>
```

## Performance Tips

1. **CSS**: Keep it inline in `<style>` tag (no external stylesheets)
2. **Images**: Use data URIs or font icons instead of external images
3. **JavaScript**: Keep logic simple, avoid large libraries
4. **Loops**: Limit displayed items for very large proxy lists:
   ```html
   {{range first 50 .result}}
       {{.}}
   {{end}}
   ```

## Examples

### Display Traffic with Bars
```html
<div style="width: 200px; height: 20px; background: #e0e0e0; border-radius: 10px;">
    <div style="width: {{div (mul .uploadByte 100) .totalByte}}%; height: 100%; background: #4CAF50; border-radius: 10px;"></div>
</div>
```

### Show Expiry Status
```html
{{if eq .expire 0}}
    <span style="color: green;">✓ No expiry</span>
{{else if gt .expire 1735689600}}
    <span style="color: green;">✓ Active</span>
{{else}}
    <span style="color: red;">✗ Expired</span>
{{end}}
```

### Format Timestamp to Date
```html
<!-- Note: Raw timestamps need JavaScript formatting -->
<script>
    const expire = {{.expire}};
    const date = new Date(expire * 1000);
    document.write(date.toLocaleDateString());
</script>
```

## Reverting to Built-in Template

Simply delete or rename the custom template:
```bash
sudo rm /etc/x-ui/templates/sub.html
```

The next request will use the built-in template automatically.

## Support

For issues or questions:
1. Check the logs: `sudo journalctl -u x-ui -n 50`
2. Verify template syntax in Go template documentation
3. Test with one of the provided templates first
