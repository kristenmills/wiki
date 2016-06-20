= Generic series features =

'''GOAL:''' Extract nice functionalities like timeframe, quality upgrades etc into separate plugins that can be re-used with movies.

'''Feature prospects'''

* Timeframe
* Propers
* Qualities

Disclaimer: Some of this might not be applicable, it has been long time since I've done anything to the codebase.

Each of these sub-functions could be separate plugins/services. Let's think timeframe for example.


{{{
class Timeframe:
  def evaluate(self, entry, identified_by, config):
     .
     .
     # return None or entry that should be accepted
}}}


Example usage from series_plugin

{{{
accepted = plugin.get_plugin_by_name('timeframe').instance.evaluate(
    entry, "{0} {1}".format(series_name, series_id, timeframe_config)
}}}


Same approach would be used for all features. Some of the plugins would need to keep track of entries internally. Currently a backlog plugin is used to keep items in the task until timeframe for example does it's thing.

Ps. As an addition to timeframe I would like to set "deadline" for series, get something regardless of the quality before certain time.

=== Generic use in task ===

This could be used in task standalone

{{{
timeframe:
  identified_by: {{ series_name }} {{ series_id }}
  target: 720p
}}}


=== Standalone with movies ===

{{{
rss: .....
imdb_lookup: yes
timeframe: 
  amount: 12h
  target: 720p
  identified_by: {{ imdb_id }}
imdb:
  min_score: 6.5
}}}
 
'''NOTES:''' 

* I hate how identified_by NEEDS to use jinja2 formatting, it makes no sense for enduser
* Timeframe needs to operate on accepted entries in order to work well with other plugins

=== With new movies plugin ===

{{{
rss: ....
movies:
  from:
    - movie_list: movies
  timeframe:
    amount: 12h
    target: 720p
}}}

'''NOTES:''' 

* movies plugin will need to supply identified_by to timeframe somehow
* does movies plugin bring anything here opposed to

{{{
rss: ...
imdb_lookup: yes
list_match:
  from:
    - movie_list: movies
timeframe:
  identified_by: {{ imdb_id }}
  amount: 12h
  target: 720p
}}}

== identified_by ==

Maybe we can get rid of having to specify identified_by manually by providing some facilities for that.

Issue comes down to now knowing which fields are id fields and which are not. Possible solutions:

'''Option 1'''

{{{
entry['ids'] = ['imdb_id', 'tmdb_id']
}}}

appended by imdb_lookup etc


'''Option 2'''

Use type

{{{
class Identity:
  
   def __init__(self, name, value):
     self.name = name
     self.value = value
}}}

usage:

{{{
entry['imdb_id'] = Identity('imdb', 123)

isinstance(ntry['imdb_id'], Identity)
}}}