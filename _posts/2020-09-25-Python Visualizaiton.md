---
layout:     post
title:      python visualization
subtitle:   
date:       2020-09-25
author:     neverset
header-img: img/post-bg-kuaidi.jpg
catalog: true
tags:
    - visualization
---

## Plotly Express
By default, all of the Plotly visualizations are interactive

### basic functions

    import plotly.express as px
    px.scatter()
    px.line()
    px.bar()
    px.histogram()
    px.box()
    px.violin()
    px.strip()

### many chart types

    fig = px.treemap()
    fig = px.sunburst(
    #to save image
    fig.write_image()
    #save to html
    fig.write_html()

### integrate with pandas

    pd.options.plotting.backend = "plotly"
    fig = df[['sodium', 'potass']].plot(kind='hist',
                                    nbins=50,
                                    histnorm='probability density',
                                    opacity=0.75,
                                    marginal='box',
                                    title='Potassium and Sodium Distributions')
    fig.write_image('potassium_sodium_plots.png')

### Interactive options

#### graph_obj
create interactive map

    # Manipulating the original dataframe 
    df_countrydate = df_countries.groupby(['Date','Country']). sum().reset_index() 
    #Creating the visualization 
    fig = px.choropleth(df_countrydate, locations="Country", locationmode = "country names", color="AvTemp", hover_name="Country", animation_frame="Date" ) 
    fig.update_layout( title_text = 'Average Temperature Change', title_x = 0.5, geo=dict( showframe = False, showcoastlines = False, )) 
    fig.show()

#### Dash
built based on Flask (as backend), React(as frontend), and Plotly, ideal for building visualization apps.
Features:
* easy to build complicated UIs with multiple inputs, multiple outputs, and inputs that depend on other inputs
* support multi-user with independent sessions
drawbacks: 
* no intermediate values in the reactive graph
* concurrency in callback is not possible
* Cannot have callbacks with no Inputs or with no Output

##### structure

|-Assets
|-Components
|-Layout
|-Pages
|-Utils
|-Environments 
|-Cache-Directory

1) Assets
Dash will automatically serve files (mages, Stylesheets, or Javascript files) in this folder
2) Components
reusable components

    import dash_html_components as html
    from utils.functions import formatter_2_decimals

    def make_dash_table(df):
        body = []
        header = []
        for column in df.columns:
            header.append(html.Th(column))

        for index, row in df.iterrows():
            html_row = []
            for i in range(len(row)):
                html_row.append(html.Td(formatter_2_decimals(row[i])))
            body.append(html.Tr(html_row))
        
        tHead = html.Thead(html.Tr(header))
        tBody = html.Tbody(body)
        table = html.Table([tHead, tBody])

        return table

3) Layout
common layout including UI view and callback functions

    #layout.py
    import dash_html_components as html
    import dash_core_components as dcc
    from layout.sidebar.sidebar import sidebar
    content = html.Div(id="page-content")
    layout = html.Div([dcc.Location(id="url"), sidebar, content])

    #view.py
    import dash_bootstrap_components as dbc
    import dash_html_components as html
    from utils.constants import home_page_location, gdp_page_location, iris_page_location
 
    # sidebar header consists of a title, and a toggle, the latter is hidden on large screens  
    sidebar_header = dbc.Row(
        [
            dbc.Col(html.H2("Sidebar", className="display-4")),
            dbc.Col(
                [
                    html.Button(
                        # use the Bootstrap navbar-toggler classes to style
                        html.Span(className="navbar-toggler-icon"),
                        className="navbar-toggler",
                        # the navbar-toggler classes don't set color
                        style={
                            "color": "rgba(0,0,0,.5)",
                            "border-color": "rgba(0,0,0,.1)",
                        },
                        id="navbar-toggle",
                    ),
                    html.Button(
                        # use the Bootstrap navbar-toggler classes to style
                        html.Span(className="navbar-toggler-icon"),
                        className="navbar-toggler",
                        # the navbar-toggler classes don't set color
                        style={
                            "color": "rgba(0,0,0,.5)",
                            "border-color": "rgba(0,0,0,.1)",
                        },
                        id="sidebar-toggle",
                    ),
                ],
                # the column containing the toggle will be only as wide as the
                # toggle, resulting in the toggle being right aligned
                width="auto",
                # vertically align the toggle in the center
                align="center",
            ),
        ]
    )

    sidebar = html.Div(
        [
            sidebar_header,
            # we wrap the horizontal rule and short blurb in a div that can be
            # hidden on a small screen
            html.Div(
                [
                    html.Hr(),
                    html.P(
                        "A responsive sidebar layout with collapsible navigation "
                        "links.",
                        className="lead",
                    ),
                ],
                id="blurb",
            ),
            # use the Collapse component to animate hiding / revealing links
            dbc.Collapse(
                dbc.Nav(
                    [
                        dbc.NavLink("Home", href=home_page_location, active="exact"),
                        dbc.NavLink("GDP", href=gdp_page_location, active="exact"),
                        dbc.NavLink("Iris", href=iris_page_location, active="exact"),
                    ],
                    vertical=True,
                    pills=True,
                ),
                id="collapse",
            ),
        ],
        id="sidebar",
    )

    #callback.py
    from dash.dependencies import Input, Output, State
    from app import app

    @app.callback(
        Output("sidebar", "className"),
        [Input("sidebar-toggle", "n_clicks")],
        [State("sidebar", "className")],
    )
    def toggle_classname(n, classname):
        if n and classname == "":
            return "collapsed"
        return ""

    @app.callback(
        Output("collapse", "is_open"),
        [Input("navbar-toggle", "n_clicks")],
        [State("collapse", "is_open")],
    )
    def toggle_collapse(n, is_open):
        if n:
            return not is_open
        return is_open    

4) Pages
MVC architectural pattern for each page

    #model.py
    import pandas as pd
    from app import cache
    from utils.constants import TIMEOUT

    @cache.memoize(timeout=TIMEOUT)
    def query_data():
        # This could be an expensive data querying step
        gdp_data = pd.read_csv('https://raw.githubusercontent.com/plotly/datasets/master/gapminderDataFiveYear.csv')
        return gdp_data.to_json(date_format='iso', orient='split')

    def dataframe():
        return pd.read_json(query_data(), orient='split')

    #view.py
    import dash_core_components as dcc
    import dash_html_components as html
    from pages.gdp.gdp_data import dataframe

    layout = html.Div([
        html.H1("GDP viewer"),
        html.Hr(),
        dcc.Graph(id='graph-with-slider'),
        dcc.Slider(
            id='year-slider',
            min=dataframe()['year'].min(),
            max=dataframe()['year'].max(),
            value=dataframe()['year'].min(),
            marks={str(year): str(year) for year in dataframe()['year'].unique()},
            step=None
        )
    ])

    #controller.py
    from dash.dependencies import Input, Output
    import plotly.express as px
    from app import app
    from pages.gdp.gdp_data import dataframe

    @app.callback(
        Output('graph-with-slider', 'figure'),
        Input('year-slider', 'value'))
    def update_figure(selected_year):
        gdp_data = dataframe()
        filtered_df = gdp_data[gdp_data.year == selected_year]

        fig = px.scatter(filtered_df, x="gdpPercap", y="lifeExp",
                        size="pop", color="continent", hover_name="country",
                        log_x=True, size_max=55)

        fig.update_layout(transition_duration=500)

        return fig


5) Utils
functions can be used for general purposes

6) Environments

    #setting dev and prod environments
    import os
    from os.path import join, dirname
    from dotenv import load_dotenv

    dotenv_path = join(dirname(__file__), os.getenv("ENVIRONMENT_FILE"))
    load_dotenv(dotenv_path=dotenv_path, override=True)

    APP_HOST = os.environ.get("HOST")
    APP_PORT = int(os.environ.get("PORT"))
    APP_DEBUG = bool(os.environ.get("DEBUG"))
    DEV_TOOLS_PROPS_CHECK = bool(os.environ.get("DEV_TOOLS_PROPS_CHECK"))

7) Cache-Directory
use caches to store intermiediate results

    from flask_caching import Cache
    cache = Cache(app.server, config={
        'CACHE_TYPE': 'filesystem',
        'CACHE_DIR': 'cache-directory'
    })

8) Routing

    #route.py
    import dash_bootstrap_components as dbc
    import dash_html_components as html
    from dash.dependencies import Input, Output

    from app import app

    from utils.constants import home_page_location, gdp_page_location, iris_page_location

    from pages.home import home
    from pages.gdp import gdp
    from pages.iris import iris

    @app.callback(
    Output("page-content", "children"), 
    Input("url", "pathname")
    )
    def render_page_content(pathname):
        if pathname == home_page_location:
            return home.layout
        elif pathname == gdp_page_location:
            return gdp.layout
        elif pathname == iris_page_location:
            return iris.layout
        # If the user tries to reach a different page, return a 404 message
        return dbc.Jumbotron(
            [
                html.H1("404: Not found", className="text-danger"),
                html.Hr(),
                html.P(f"The pathname {pathname} was not recognised..."),
            ]
        )

9) index.py and app.py

    #app
    import dash
    import dash_bootstrap_components as dbc
    from flask_caching import Cache
    from utils.external_assets import FONT_AWSOME, CUSTOM_STYLE
    from layout.layout import layout
    import flask

    server = flask.Flask(__name__) # define flask app.server
    app = dash.Dash(
        __name__,
        server=server,
        suppress_callback_exceptions=True, 
        external_stylesheets=[
            dbc.themes.BOOTSTRAP,
            FONT_AWSOME,
            CUSTOM_STYLE
        ],
        meta_tags=[
            {"name": "viewport", "content": "width=device-width, initial-scale=1"}
        ]
    )
    cache = Cache(app.server, config={
        'CACHE_TYPE': 'filesystem',
        'CACHE_DIR': 'cache-directory'
    })

    app.layout = layout

    #index.py
    from app import app, server
    from routes import render_page_content
    from layout.sidebar.sidebar_callbacks import toggle_collapse, toggle_classname
    from pages.gdp.gdp_callbacks import update_figure
    from pages.iris.iris_callbacks import make_graph
    from environment.settings import APP_HOST, APP_PORT, APP_DEBUG, DEV_TOOLS_PROPS_CHECK

    if __name__ == "__main__":
        app.run_server(
            host=APP_HOST,
            port=APP_PORT,
            debug=APP_DEBUG,
            dev_tools_props_check=DEV_TOOLS_PROPS_CHECK
        )



#### Streamlit
streamlit run streamlit_example.py

    #streamlit_example.py
    import streamlit as st
    import pandas as pd
    import plotly.express as px


    @st.cache()
    def load_data():
        df = pd.read_csv(
            'https://github.com/chris1610/pbpython/blob/master/data/cereal_data.csv?raw=True'
        )
        return df


    # Read in the cereal data
    df = load_data()

    st.title('Rating exploration')

    # Only a subset of options make sense
    x_options = [
        'calories', 'protein', 'fat', 'sodium', 'fiber', 'carbo', 'sugars',
        'potass'
    ]

    # Allow use to choose
    x_axis = st.sidebar.selectbox('Which value do you want to explore?', x_options)

    # plot the value
    fig = px.scatter(df,
                    x=x_axis,
                    y='rating',
                    hover_name='name',
                    title=f'Cereal ratings vs. {x_axis}')

    st.plotly_chart(fig)

## Datapane
Datapane is an API allow people to view visualization result and share result easily in the cloud.Deploying Python scripts and notebooks is also possible.  
It supports:
* Pandas  
* Plots from Python visualization libraries such as Plotly, Bokeh, Altair, and Folium
* Markdown
To install

    pip3 install datapane
    #generate report you need token
    datapane login --server=https://datapane.com/ --token=yourtoken

### generate report

    import pandas as pd
    import altair as alt
    import datapane as dp
    df = pd.read_csv('https://query1.finance.yahoo.com/v7/finance/download/GOOG?period2=1585222905&interval=1mo&events=history')
    chart = alt.Chart(df).encode(
        x='Date:T',
        y='Open'
    ).mark_line().interactive()
    #generate report
    r = dp.Report(
    dp.Markdown('My simple report'), #add description to the report
    dp.Table(df), #create a table
    dp.Plot(chart) #create a chart
    )

    # Publish your report. Make sure to have visibility='PUBLIC' if you want to share your report
    r.publish(name='stock_report', visibility='PUBLIC')
        
### add interactive parameter in report 

    datapane script deploy --script=<your_python_script.py> --name=name_of_your_report

## voila
voila can convert jupyter script into a standalone web application, it support ipywidgets library in jupyter

    $ pip install voila
    $ conda install voila -c conda-forge
    #convert command
    voila <jupyter_script> <command-line options>
    #for example
    voila test.ipynb --theme=dark --template=material

### Create interactive widgets

#### slider

    import ipywidgets as widgets
    import pandas as pd
    style = {'description_width': 'initial'}
    limit_case = widgets.IntSlider(
        value=1000,
        min=100,
        max=5000,
        step=1,
        description='Max Number of Case:',
        disabled=False,
        style=style
    )

    def update_df_length(limit):   
        df = pd.read_csv('SF_crimes.csv')
        df = df.iloc[0:limit, :]
        print("Number of rows in the dataset that have been successfully loaded:"+str(len(df)))

    #connect widgets with callback function
    widgets.interactive(update_df_length, limit=limit_case)

#### multiple selection

    import pandas as pd
    import ipywidgets as widgets
    from ipywidgets import Layout
    df = pd.read_csv('SF_crimes.csv')
    unique_district = df.PdDistrict.unique()
    district = widgets.SelectMultiple(
        options = unique_district.tolist(),
        value = ['BAYVIEW', 'NORTHERN'],
        description='District',
        disabled=False,
        layout = Layout(width='50%', height='80px', display='flex')
    )
    district

#### intergrate widgets with map

    import matplotlib.pyplot as plt
    from folium import plugins

    def update_map(district, category, limit):
        
        df = pd.read_csv('SF_crimes.csv')
        df = df.iloc[0:limit, :]
        
        latitude = 37.77
        longitude = -122.42
        
        df_dist = df.loc[df['PdDistrict'].isin(np.array(district))]
        df_category = df_dist.loc[df_dist['Category'].isin(np.array(category))]
        
        cat_unique = df_category['Category'].value_counts()
        cat_unique = cat_unique.reset_index()
        
        dist_unique = df_category['PdDistrict'].value_counts()
        dist_unique = dist_unique.reset_index()
        
        fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(20,10))

        # create map and display it
        sanfran_map = folium.Map(location=[latitude, longitude], zoom_start=12)

        
        incidents = plugins.MarkerCluster().add_to(sanfran_map)

    # loop through the dataframe and add each data point to the mark cluster
        for lat, lng, label, in zip(df_category.Y, df_category.X, df_category.Category):
            folium.Marker(
            location=[lat, lng],
            icon=None,
            popup=label,
            ).add_to(incidents)
    # show map
        display(sanfran_map)
        
        ax1.bar(cat_unique['index'], cat_unique['Category'])
        ax1.set_title('Amount of Criminal Case Based on Category')
        ax2.bar(dist_unique['index'], dist_unique['PdDistrict'])
        ax2.set_title('Amount of Criminal Case in Selected District')
        
        plt.show()

## matplotlib
### FuncAnimation
create dynamic plot with animator = ani.FuncAnimation(fig, chartfunc, interval = 100)

    import matplotlib.animation as ani
    import numpy as np
    import matplotlib.pyplot as pltcolor = ['red', 'green', 'blue', 'orange']
    fig = plt.figure()
    plt.xticks(rotation=45, ha="right", rotation_mode="anchor") #rotate the x-axis values
    plt.subplots_adjust(bottom = 0.2, top = 0.9) #ensuring the dates (on the x-axis) fit in the screen
    plt.ylabel('No of Deaths')
    plt.xlabel('Dates')

    #define chartfunc
    def buildmebarchart(i=int):
        plt.legend(df1.columns)
        p = plt.plot(df1[:i].index, df1[:i].values) #note it only returns the dataset, up to the point i
        for i in range(,4):
            p[i].set_color(color[i]) #set the colour of each curveimport matplotlib.animation as ani

    #call funcanimation to make plot dynamic
    animator = ani.FuncAnimation(fig, buildmebarchart, interval = 100)
    plt.show()

### Using LaTeX font

    # import libraries
    import numpy as np
    import matplotlib.pyplot as plt
    # adjust matplotlib parameters
    plt.rcParams.update(plt.rcParamsDefault)
    plt.rcParams['text.usetex'] = True
    plt.rcParams['font.size'] = 18
    plt.rcParams['font.family'] = "serif"
    # create mock data
    r = 15
    theta = 5
    rw = 12
    gamma = 0.1
    err = np.arange(0., r, .1)
    z = np.where(err < rw, 0, gamma * (err-rw)**2 * np.sin(np.deg2rad(theta)))  
    # visualize the data
    plt.scatter(err, z, s = 10)
    plt.title(r'$\Sigma(x) = \gamma x^2 \sin(\theta)$', pad = 20)
    plt.xlabel('x')
    plt.ylabel('$\phi$')
    # save the plots
    plt.savefig('latex.png', dpi = 300, pad_inches = .1, bbox_inches = 'tight')

### zoom-in effect

    # create one zoom in
    fig = plt.figure()
    # Set the random seed 
    np.random.seed(100)
    # Create mock data
    x = np.random.normal(400, 50, 10_000)
    y = np.random.normal(300, 50, 10_000)
    c = np.random.rand(10_000)
    # Create zoom-in plot
    ax = plt.scatter(x, y, s = 5, c = c)
    plt.xlim(400, 500)
    plt.ylim(350, 400)
    plt.xlabel('x', labelpad = 15)
    plt.ylabel('y', labelpad = 15)
    # Create zoom-out plot
    ax_new = fig.add_axes([0.6, 0.6, 0.2, 0.2]) # the position of zoom-out plot compare to the ratio of zoom-in plot 
    plt.scatter(x, y, s = 1, c = c)
    # Save figure with nice margin
    plt.savefig('zoom.png', dpi = 300, bbox_inches = 'tight', pad_inches = .1)

    #create two zoom in 
    # Define a function using lambda
    stock = lambda A, amp, angle, phase: A * angle + amp * np.sin(angle + phase)
    # Define parameters 
    theta = np.linspace(0., 2 * np.pi, 250) # x-axis
    np.random.seed(100)
    noise = 0.2 * np.random.random(250)
    y = stock(.1, .2, theta, 1.2) + noise # y-axis
    # Create main container with size of 6x5
    fig = plt.figure(figsize=(6, 5))
    plt.subplots_adjust(bottom = 0., left = 0, top = 1., right = 1)
    # Create first axes, the top-left plot with green plot
    sub1 = fig.add_subplot(2,2,1) # two rows, two columns, fist cell
    sub1.plot(theta, y, color = 'green')
    sub1.set_xlim(1, 2)
    sub1.set_ylim(0.2, .5)
    sub1.set_ylabel('y', labelpad = 15)
    # Create second axes, the top-left plot with orange plot
    sub2 = fig.add_subplot(2,2,2) # two rows, two columns, second cell
    sub2.plot(theta, y, color = 'orange')
    sub2.set_xlim(5, 6)
    sub2.set_ylim(.4, 1)
    # Create third axes, a combination of third and fourth cell
    sub3 = fig.add_subplot(2,2,(3,4)) # two rows, two colums, combined third and fourth cell
    sub3.plot(theta, y, color = 'darkorchid', alpha = .7)
    sub3.set_xlim(0, 6.5)
    sub3.set_ylim(0, 1)
    sub3.set_xlabel(r'$\theta$ (rad)', labelpad = 15)
    sub3.set_ylabel('y', labelpad = 15)
    # Create blocked area in third axes
    sub3.fill_between((1,2), 0, 1, facecolor='green', alpha=0.2) # blocked area for first axes
    sub3.fill_between((5,6), 0, 1, facecolor='orange', alpha=0.2) # blocked area for second axes
    # Create left side of Connection patch for first axes
    con1 = ConnectionPatch(xyA=(1, .2), coordsA=sub1.transData, 
                        xyB=(1, .3), coordsB=sub3.transData, color = 'green')
    # Add left side to the figure
    fig.add_artist(con1)
    # Create right side of Connection patch for first axes
    con2 = ConnectionPatch(xyA=(2, .2), coordsA=sub1.transData, 
                        xyB=(2, .3), coordsB=sub3.transData, color = 'green')
    # Add right side to the figure
    fig.add_artist(con2)
    # Create left side of Connection patch for second axes
    con3 = ConnectionPatch(xyA=(5, .4), coordsA=sub2.transData, 
                        xyB=(5, .5), coordsB=sub3.transData, color = 'orange')
    # Add left side to the figure
    fig.add_artist(con3)
    # Create right side of Connection patch for second axes
    con4 = ConnectionPatch(xyA=(6, .4), coordsA=sub2.transData, 
                        xyB=(6, .9), coordsB=sub3.transData, color = 'orange')
    # Add right side to the figure
    fig.add_artist(con4)
    # Save figure with nice margin
    plt.savefig('zoom_effect_2.png', dpi = 300, bbox_inches = 'tight', pad_inches = .1)

### create outbox legend

    plt.legend(bbox_to_anchor=(1.05, 1.04)) # position of the legend

### Creating continuous error plots

    plt.fill_between(x, upper_limit, lower_limit)

### Adjusting box pad margin
using plt.savefig() followed by a complex argument: bbox_inches and pad_inches

## Bokeh
Bokeh is a python library for interactive data visualization and web app
### Glyphs
  
    from bokeh.io import output_file, show
    from bokeh.plotting import figure

    plot = figure()
    plot.line(x=[1,2,3,4], y=[4,7,2,16])
    output_file('line_glyph.html')
    plot.circle([1,2,3,4], [4,7,2,16], fill_color="white", size=10)
    show(plot)
### Data Sources format
compatible with data formats of lists, NumPy arrays and pandas DataFrames

#### ColumnDataSource
inbuilt data format in bokeh is ColumnDataSource. access data is by specifying its column name, and passing our CDS as the source parameter of a plotting method

    
    from bokeh.models import ColumnDataSource
    source_from_dict = ColumnDataSource(data={
                                    'x' : [1,2,3,4,5],
                                    'y' : [1,4,9,16,25]})
    #or
    source_from_df = ColumnDataSource(df)
    plot = figure()
    plot.line(x='Date', y='Open', source=source)
    output_file('cds_glyph.html')
    show(plot)

### customization
#### customize plot tool

    from bokeh.plotting import figure, show
    p = figure(toolbar_location="left",
        tools="pan,wheel_zoom,lasso_select,tap,undo,reset")
    show(p)

#### HoverTools
Field names are prepended with @, plot metadata are prepened with $

    from bokeh.models import HoverTool
    hover = HoverTool(tooltips=[
            ("index", "$index"),
            ("label", "@column_name"),
            ("label2", "@{multi-word column name}"),
            ("colour", "$color[hex, swatch]:fill_color")])

#### color mapping

    mapper = linear_cmap(field_name='y', palette=Spectral6, low=min(y), high=max(y))
    plot = figure(plot_height=300, plot_width=600)
    plot.circle(x,y, color=mapper, line_color='black', size=10)

#### annotations

    p = figure(
        plot_width=500,
        plot_height=500,
        x_axis_label="X Label",
        y_axis_label="Y Label",
        title="My custom plot",
        toolbar_location="left",
        tools="pan,wheel_zoom,reset")
    show(p)

#### arrange plots

    layout = row(p1, p2, p3)
    #or 
    layout = column(p1, p2, p3)
    #or
    layout = gridplot([None, p1], [p2, p3], toolbar_location=None)
    # add tab
    first = Panel(child=row(p1, p2), title='first')
    second = Panel(child=row(p3), title='second')
    tabs = Tabs(tabs=[first, second])
    # link plot with same data source
    p3.x_range = p2.x_range = p1.x_range
    p3.y_range = p2.y_range = p1.y_range
    show(layout)

## Pygal

    #pip install pygal
    import pygal
    bar_chart = pygal.Bar() 
    bar_chart.title = 'NBA data'

    #add data
    bar_chart.add('qw', 38387) 
    bar_chart.add('wq', 36928) 
    bar_chart.add('eq', 34384) 
    bar_chart.add('eqw', 33643) 
    bar_chart.add('fsdf', 32292) 
    bar_chart.render_in_browser()
    bar_chart.render_to_file('NBA.svg')




