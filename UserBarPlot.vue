<template lang="jade">
svg(width="100%" v-bind:viewBox="chartData.viewBox")
  g(v-bind:transform="chartData.coordinateTransform")
    g
      rect.nothing.animated(
        v-bind:x="-margin"
        y=-100
        v-bind:width="daysCharted*chartData.dayWidth+2*margin"
        v-bind:height="silverAnswers*chartData.answerHeight+100"
      )
      rect.bronze.animated(
        v-bind:x="-margin"
        v-bind:y="bronzeAnswers*chartData.answerHeight"
        v-bind:width="daysCharted*chartData.dayWidth+2*margin"
        v-bind:height="(silverAnswers-bronzeAnswers)*chartData.answerHeight"
      )
      text.animated(
        font-size=10
        dy=-2
        fill="white"
        v-bind:transform="textTransform(10, bronzeAnswers*chartData.answerHeight)"
      ) {{$t('bronze')}}
      rect.silver.animated(
        v-bind:x="-margin"
        v-bind:y="silverAnswers*chartData.answerHeight"
        v-bind:width="daysCharted*chartData.dayWidth+2*margin"
        v-bind:height="(goldAnswers-silverAnswers)*chartData.answerHeight"
      )
      text.animated(
        font-size=10
        dy=-2
        fill="white"
        v-bind:transform="textTransform(10, silverAnswers*chartData.answerHeight)"
      ) {{$t('silver')}}
      rect.gold.animated(
        v-bind:x="-margin"
        v-bind:y="goldAnswers*chartData.answerHeight"
        v-bind:width="daysCharted*chartData.dayWidth+2*margin"
        v-bind:height="(goldAnswers+1000)*chartData.answerHeight"
      )
      text.animated(
        font-size=10
        dy=-2
        fill="white"
        v-bind:transform="textTransform(10, goldAnswers*chartData.answerHeight)"
      ) {{$t('gold')}}
    g(v-for="bar in chartData.bars")
      rect.success.fill.animated(
        v-bind:x="bar.x"
        y=0
        v-bind:width="chartData.dayWidth"
        v-bind:height="bar.greenHeight"
      )
      rect.error.fill.animated(
        v-bind:x="bar.x"
        v-bind:y="bar.greenHeight"
        v-bind:width="chartData.dayWidth"
        v-bind:height="bar.redHeight"
      )
      text.animated(
        v-if="chartData.answerHeight * bar.correctCount > 5"
        dy=5
        font-size=5
        fill="white"
        text-anchor="middle"
        v-bind:transform="textTransform(bar.x + chartData.dayWidth / 2 , bar.greenHeight)"
      ) {{bar.correctCount}}
      text(
        dy=3.5
        font-size=3.5
        fill="white"
        text-anchor="middle"
        v-bind:transform="textTransform(bar.x + chartData.dayWidth / 2 , 0)"
      ) {{bar.label}}
</template>

<script lang="coffee">
import { UserStatistics } from "/imports/api/userStatistics.coffee"
import _ from "lodash"
return
  data : ->
    daysCharted : 7
    dayGap : 2
    grafWidth : 200
    grafHeight : 100
    margin : 10
    bronzeAnswers : 10
    silverAnswers : 50
    goldAnswers : 100
    userStatistics : {}
  methods :
    textTransform : (x, y) ->
      "scale(1 -1) translate(#{x} #{-y})"
  computed :
    chartData : ->
      keyFormat="D-M-Y"
      submissions = @userStatistics?.submissions
      dayData = [@daysCharted-1..0].map (daysAgo) =>
        date =
          moment()
          .subtract daysAgo, "days"
          .startOf("day")
          .format(keyFormat)
        label =
          moment()
          .subtract daysAgo, "days"
          .startOf("day")
          .calendar(null,
            sameDay: @$t 'heute'
            nextDay: @$t 'morgen'
            nextWeek: 'dd'
            lastDay: @$t 'gestern'
            lastWeek: 'dd'
            sameElse: @$t 'D_M'
          )
        thatDay = submissions?.byDate?[date]
        correctCount = thatDay?.correct or 0
        falseCount = thatDay?.incorrect or 0
        totalCount = thatDay?.total or 0
        {date, label,daysAgo, correctCount, falseCount, totalCount}
      maxDayTotal = (_.maxBy dayData, "totalCount").totalCount
      maxAnswers = switch
        when maxDayTotal < 1 then @bronzeAnswers - 1
        when maxDayTotal < @bronzeAnswers then @bronzeAnswers + 2
        when maxDayTotal < @silverAnswers then @silverAnswers + 2
        when maxDayTotal < @goldAnswers then @goldAnswers + 2
        else maxDayTotal + 2
      dayWidth = @grafWidth/@daysCharted
      answerHeight = @grafHeight/maxAnswers
      viewBox = "#{-@margin} #{-@margin} #{@grafWidth+2*@margin} #{@grafHeight+2*@margin}"
      coordinateTransform = "translate(0 100) scale(1 -1)"
      bars = dayData.map (day, index) =>
        x : dayWidth * index + @dayGap/2
        greenHeight : answerHeight * day.correctCount
        redHeight : answerHeight * day.falseCount
        correctCount : day.correctCount
        label : day.label
      return {viewBox, coordinateTransform, bars, dayWidth, answerHeight}
  meteor :
    $subscribe :
      userStatistics : -> [userId : @user?._id]
    userStatistics :
      params : -> userId : @user._id
      update : ({userId}) -> UserStatistics.findOne(userId)
  props :
    user :
      type : Object
      required : true
</script>

<style scoped lang="sass">
.animated
  transition: all .6s
.nothing
  fill: DimGray
.bronze
  fill: burlywood
.silver
  fill: gainsboro
.gold
  fill: #ec0
</style>
