# Team photos

Place JPG or WebP files in this folder (`assets/images/team/`). Reference them from markdown with the paths below (no leading slash).

## Team members (Individual reflections)

Use inside `{{< members >}}` on the Phase I post or any page:

```markdown
{{< members >}}
{{< member name="Ari Spokony" role="Data sources" href="/ari_spokony/blog1/" initials="AS" photo="images/team/ari.jpg" >}}
{{< member name="Bobby Bress" role="Project description" href="/bobby_bress/bress-belgium-blog/" initials="BB" photo="images/team/bobby.jpg" >}}
{{< member name="Anjali Patel" role="Personas & user stories" href="/anjali_patel/anjali-intro/" initials="AP" photo="images/team/anjali.jpg" >}}
{{< member name="Rayna Patel" role="Personas & user stories" href="/rayna_patel/rayna-intro/" initials="RP" photo="images/team/rayna.jpg" >}}
{{< /members >}}
```

Omit `photo="..."` until the file exists; initials show as a fallback.

## Personas (user research)

```markdown
{{< persona name="Lena Müller" role="Household Owner" age="30" location="Hamburg, Germany" initials="LM" type="household" photo="images/team/Lena.jpg" >}}
...
{{< /persona >}}

{{< persona name="Marco Frite" role="Energy Journalist" age="30" location="Brussels, Belgium" initials="MF" type="journalist" photo="images/team/marco.jpg" >}}
...
{{< /persona >}}

{{< persona name="Sofia Anderson" role="Policy Analyst" age="47" location="Brussels, Belgium" initials="SA" type="policy" photo="images/team/sofia.jpg" >}}
...
{{< /persona >}}
```

## Figure shortcode (blog posts)

```markdown
{{< figure src="images/team/ari.jpg" alt="Ari Spokony" caption="Week one in Leuven" >}}
```

Note: `figure` expects a full URL or path Hugo can resolve; for processed assets, prefer the `member` or `persona` shortcodes above.

## Current files

| File | Used for |
|------|----------|
| `ari.jpg` | Ari Spokony |
| `bobby.jpg` | Bobby Bress |
| `rayna.jpg` | Rayna Patel |
| `anjali.jpg` | Anjali Patel (add when ready) |
| `Lena.jpg` | Persona: Lena Müller |
| `marco.jpg` | Persona: Marco Frite |
| `sofia.jpg` | Persona: Sofia Anderson |
