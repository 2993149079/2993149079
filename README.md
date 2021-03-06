library(Tetris)
library(beepr)
fullTable<-totalMatrix()
cubes<-GnrCubes()
Gameon<-FALSE
server <- function(input, output,session) {
  totalscore<-0
  bgtable <-drawTable()
  active<-reactiveVal(FALSE)
  observeEvent(input$pressedKey,{
    if (!is.null(input$keyPressed) && Gameon)
    {
      active(FALSE)
      code<-input$keyPressed
      
      if(code==37) ##Press Left Arrow
      {
        cubes<<-MoveLeft(cubes,fullTable)
        beep(2)
      }
      if(code==39) ##Press Right Arrow
      {
        cubes<<-MoveRight(cubes,fullTable)
        beep(2)
      }
      if(code==32) ##Press Spacebar
      {
        cubes<<-MoveDown(cubes,fullTable)
        beep(2)
      }
      if(code==87) ##Press W
      {
        cubes<<-rotate(cubes,fullTable)
        #cubes<<-MoveRight(cubes)
      }
      active(TRUE)
    }
  })
  
  observe(
    {
      invalidateLater(1500, session)
      isolate({
        if(active())
        {
          bt<-UpdateTable(bgtable,cubes$cubesID)
          continueDrop<-checkNextBlock_y(cubes$cubesID,fullTable)
          if(continueDrop)
          {
            cubes$cubesID[,"y"]<<-cubes$cubesID[,"y"]-1
            rownames(cubes$cubeMatrix)<<-as.numeric(rownames(cubes$cubeMatrix))-1
          }
          else
          {
            for (i in 1:nrow(cubes$cubesID))
            {
              if(cubes$cubesID[i,"y"]>20)
                next()
              fullTable[as.character(cubes$cubesID[i,"y"]),as.character(cubes$cubesID[i,"x"])]<<-1
            }
            score<-GetScore(fullTable)
            if(score$score>0)
            {
              fullTable<<-score$tables
              totalscore<<-totalscore+score$score
              {
                output$ScorePanel <- renderText({paste0("Score: ",totalscore)   })
              }
            }
            bgtable<<-updateBackGround(fullTable)
            if(endGame(fullTable))
            {
              active(FALSE)
              Gameon<<-FALSE
              output$LevelInfo<-renderText("Game Over")
            }
            cubes<<-GnrCubes()
            #active(FALSE)
          }
          output$plot <- renderPlot({
            bt
          })
        }
      })
    })
  
  
  output$plot <- renderPlot({
    bgtable
  })
  output$currentTime <- renderText({
    invalidateLater(1000, session)
    paste("Time: ", Sys.time())
  })
  output$LevelInfo<-renderText("Level 1")
  output$ScorePanel <- renderText({"Score: 0"  })
  observeEvent(input$startGame,{active(TRUE)
    fullTable<<-totalMatrix()
    cubes<<-GnrCubes()
    Gameon<<-TRUE
    bgtable <<-drawTable()})
  observeEvent(input$endGame,{
    active(FALSE)
    Gameon<<-FALSE
  })
  observeEvent(input$reset,{active(FALSE)
    output$LevelInfo<-renderText("Level 1")
    cubes<<-GnrCubes()
    bgtable <<-drawTable()
    output$plot <- renderPlot({
      bgtable
    })})
}






