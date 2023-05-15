---
# Leave the homepage title empty to use the site title
title:
date: 2022-10-24
type: landing

sections:
  - block: about.biography
    id: about
    content:
      title: Biography
      # Choose a user profile to display (a folder name within `content/authors/`)
      username: admin
  - block: features
    content:
      title: Skills
      items:
        - name: "Operating Systems"
          icon: "linux"
          icon_pack: fab
        - name: "Critical Systems"
          icon: "plane"
          icon_pack: fas
        - name: "Programming"
          icon: "laptop-code"
          icon_pack: fas
        - name: "Communication Networks"
          icon: "network-wired"
          icon_pack: fas
        - name: "Versionning"
          icon: "code-branch"
          icon_pack: fas
        - name: 'Design'
          icon: "project-diagram"
          icon_pack: fas
        - name: 'Automation'
          icon: "robot"
          icon_pack: fas
        - name: 'Integrating'
          icon: cogs
          icon_pack: fas
        - name: 'Teaching'
          icon: "graduation-cap"
          icon_pack: fas
  - block: experience
    content:
      title: Experience
      # Date format for experience
      #   Refer to https://wowchemy.com/docs/customization/#date-format
      date_format: Jan 2006
      # Experiences.
      #   Add/remove as many `experience` items below as you like.
      #   Required fields are `title`, `company`, and `date_start`.
      #   Leave `date_end` empty if it's your current employer.
      #   Begin multi-line descriptions with YAML's `|2-` multi-line prefix.
      items:
        - title: "Software Engineer"
          company: "SoundHound, Inc"
          company_url: "https://www.soundhound.com"
          date_end: ""
          date_start: "2021-07-21"
          date_start: "2023-07-28"
          description: ""
          location: "Paris, France"
        - title: "Software Engineer and Integration and Validation Engineer"
          company: "Thales Group"
          company_url: "https://www.thalesgroup.com/fr"
          date_start: "2019-09-02"
          date_end: "2021-07-14"
          description: |2-
              * Contribution to the **analysis** and **validation** of the implementation of a big data distributed system
              * **Identification** of problems within a defined perimeter and **reporting**
              * Responsibility for the **design** and **development** of **inspection tools**
              * **Supervision** of junior associates for the implementation of inspection tools
          location: "Toulouse, France"
        - title: "ATER (Temporary Teaching and Research Assistant), Ph.D. candidate"
          company: "ISAE-SUPAERO"
          company_url: "https://www.isae-supaero.fr/fr/"
          date_end: "2019-07-31"
          date_start: "2016-02-29T23:00:00+00:00"
          description: "Teaching assistant in C, Java, Real-time systems, SysML, and numerical analysis."
          location: "Toulouse, France"
        - title: "Software Engineer for simulation architecture, Ph.D. Candidate"
          company: "Airbus Group"
          company_url: "https://www.airbus.com/"
          date_end: "2019-02-28"
          date_start: "2016-02-29T23:00:00+00:00"
          description: |2-
              * **Formalization** of the execution of a **distributed simulation** for the *a priori* validation of a **simulation scheduling** respecting aerospatial-specific constraints  
              * **Analyzis** of existing simulations and technical documentations for formalizing of aerospatial-specific distributed simulation constraints  
              * **Implementation** of RROSACE -- a simple flight controller **case study** from ROSACE, in Matlab and C  
              * **Implementation** of seaplanes -- a **simulation framework** in C++ based on HLA, a publish-subscribe-based data exchange standard, with Qt interface
              * **Implementation** of a modulable and extensible **allocation tool** in Python, with multiple heuristics
              * **Presentation** of results and demonstrators in **international conferences**
          location: "Toulouse, France"
        - title: "Software Engineer for satellite ground segment communications"
          company: "Viveris technology"
          company_url: "https://www.viveris.fr/"
          date_end: "2016-01-31"
          date_start: "2014-11-30T23:00:00+00:00"
          description: |2-
              * **Implementation** of a DVB-RCS2 communication protocol in  satellite communications ground segment for Thalès Alenia Space / CNES
              * **Development** and **integration** of network modules in multi-threaded telecommunication **kernel device**.
              * **Development** of **Quality of Service library** for the satellite simulation environment, based on the libns
              * **Development** of **test tools** for **continuous integration** in Python, Ruby and Perl
              * **Analysis** of network packet scheduling for QoS validation"
              Open-source library available at [hnrck/librle](https://github.com/hnrck/librle)
          location: "Toulouse, France"
    design:
      columns: '2'
  - block: collection
    id: posts
    content:
      title: Recent Posts
      subtitle: ''
      text: ''
      # Choose how many pages you would like to display (0 = all pages)
      count: 5
      # Filter on criteria
      filters:
        folders:
          - post
        author: ""
        category: ""
        tag: ""
        exclude_featured: false
        exclude_future: false
        exclude_past: false
        publication_type: ""
      # Choose how many pages you would like to offset by
      offset: 0
      # Page order: descending (desc) or ascending (asc) date.
      order: desc
    design:
      # Choose a layout view
      view: compact
      columns: '2'
  - block: portfolio
    id: projects
    content:
      title: Projects
      filters:
        folders:
          - project
      # Default filter index (e.g. 0 corresponds to the first `filter_button` instance below).
      default_button_index: 0
      # Filter toolbar (optional).
      # Add or remove as many filters (`filter_button` instances) as you like.
      # To show all items, set `tag` to "*".
      # To filter by a specific tag, set `tag` to an existing tag name.
      # To remove the toolbar, delete the entire `filter_button` block.
      buttons:
        - name: All
          tag: '*'
        - name: Deep Learning
          tag: Deep Learning
        - name: Other
          tag: Demo
    design:
      # Choose how many columns the section has. Valid values: '1' or '2'.
      columns: '1'
      view: showcase
      # For Showcase view, flip alternate rows?
      flip_alt_rows: false
  - block: collection
    id: featured
    content:
      title: Featured Publications
      filters:
        folders:
          - publication
        featured_only: true
    design:
      columns: '2'
      view: card
  - block: collection
    content:
      title: Recent Publications
      text: |-
        {{% callout note %}}
        Quickly discover relevant content by [filtering publications](./publication/).
        {{% /callout %}}
      filters:
        folders:
          - publication
        exclude_featured: true
    design:
      columns: '2'
      view: citation
  - block: collection
    id: talks
    content:
      title: Recent & Upcoming Talks
      filters:
        folders:
          - event
    design:
      columns: '2'
      view: compact
  - block: tag_cloud
    content:
      title: Popular Topics
    design:
      columns: '2'
  - block: contact
    id: contact
    content:
      title: Contact
      subtitle:
      text: ""
      address:
        country: France
        country_code: FR
      autolink: true
    design:
      columns: '2'
---
